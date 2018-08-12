---
layout: post
title: 爬取微信公众号的全部文章
---

最近读六神磊磊的文章，很喜欢。微信上看他所有文章并不方便，就想全部爬下来看，之前用Python爬过很多东西了，没有使用爬虫框架，就是requests + Beautifulsoup

## 搜狗微信搜索爬取文章
这个尝试了，最终没有采用
主要问题有：

* 不登陆，只能爬取有限的文章，当然可以设置cookie，这样可以更多的文章，但是有额外的工作
* 得到的url不是最终微信文章的url，需要做二次请求，而且必须使用requests的Session，Server端有鉴权
* 反爬虫措施做的很好，我把爬取时间间隔设成10s，还是会被屏蔽
* 有的文章被和谐掉是不做备份的

## 传送门爬取
这个过程倒是没那么麻烦，大概半小时就完成了,代码大概50行左右

需要注意的是所有Http请求都要在Header里加上**User-Agent**，否则内容获取不了

```python
db_file = r"wechat_article_chuan.db"
# if os.path.exists(db_file):
#     os.remove(db_file)

conn = sa.create_engine("sqlite:///{}".format(db_file))
Base = declarative_base()

class WechatArticle(Base):
    __tablename__ = "wechat_article"
    id = sa.Column("id", sa.INTEGER, primary_key=True)
    url = sa.Column("url", sa.String)
    title = sa.Column("title", sa.String)

    def __str__(self):
        return str(self.__dict__)

Base.metadata.create_all(conn)
Session = sessionmaker(bind=conn)
session = Session()

url = r"http://chuansong.me/account/dujinyong6"
all_index_links = list()
soup = http_cache_util.get(url)
aa = crawl_util.get_links(url, ".*account.*start")
ends_page = int(aa[-1][aa[-1].rfind("=") + 1:])
print(ends_page)
i = 0
while (i-1)*12 < ends_page:
    all_index_links.append("{}?start={}".format(url, i*12))
    i = i + 1

print("\n".join(all_index_links))
for ii in all_index_links:
    soup = http_cache_util.get(ii)
    links = soup.find_all("a", {"class": "question_link"})
    for i in links:
        article_url = urljoin(url, i['href'])
        wechat_article = WechatArticle()
        wechat_article.url = article_url
        wechat_article.title = i.text.strip()
        print(wechat_article)
        session.merge(wechat_article)

session.commit()
```

**http_cache_util.get**这个函数是我做的一个把url的content缓存到本地的简单cache

**crawl_util.get_links**这个函数是我爬虫的Util类，是获取所有满足正则表达式的超链接
代码就不贴了


