# pyreptile
基于`python`的爬虫，抓取了豆瓣top250电影，并按照评分由高到低排序，写入到excel表格中。

```python
import requests
import bs4
from openpyxl import Workbook

class Movie:
    def __init__(self):
        self.title = ""
        self.rank = ""

    def setTitle(self, title):
        self.title = title

    def setRank(self, rank):
        self.rank = rank

    def __lt__(self, other):
        if self.rank > other.rank:
            return True
        else:
            return False


# 自定义排序函数
def cmp(self, other):
    if self.rank > other.rank:
        return 1
    elif self.rank == other.rank:
        return 0
    else:
        return -1


# 全局变量
movie = []


# 抓取网页
def get_page(url):
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3314.0 Safari/537.36 SE 2.X MetaSr 1.0"
    }
    r = requests.get(url, headers=headers)
    if r.status_code == 200:
        print('爬取成功')
        # print(r.text)
        return r
    else:
        print('爬取失败')
        return None


# 获取总页数
def find_depth(res):
    soup = bs4.BeautifulSoup(res.text, 'html.parser')  # 要解析的html文本 第二个参数使用那个解析器
    depth = soup.find('span', class_='next').previous_sibling.previous_sibling.text

    return int(depth)


# 解析网页内容 提取内容
def find_movies(res):
    # soup = bs4.BeautifulSoup(res.text, 'html.parser')  # 要解析的html文本 第二个参数使用那个解析器

    soup = bs4.BeautifulSoup(res.text, 'html.parser')

    # 获取电影名
    names = []
    target = soup.find_all('div', class_='hd')
    for i in target:
        names.append(i.a.span.text)

    # print(names) # 输出电影名

    # 获取评分
    ranks = []
    target = soup.find_all('span', class_='rating_num')
    for i in target:
        ranks.append(i.text)

    # ranks.sort()
    # print(ranks) # 输出评分
    length = len(ranks)

    for i in range(0, length):
        # print(ranks[i] + " " + names[i])
        m = Movie()
        m.setRank(ranks[i])
        m.setTitle(names[i])
        movie.append(m)

    rank_movies()

# 对按照评分进行排序
def rank_movies():
    # 对movie中的元素按照ranks属性进行排序
    movie.sort()
    length = len(movie)
    for i in range(0, length):
        print(movie[i].title + " " + movie[i].rank)

def write_excel():
    wb = Workbook()
    ws = wb.active
    ws.title = "豆瓣电影top250"
    ws['A1'] = "电影名"
    ws['B1'] = "评分"

    length = len(movie)

    ws.column_dimensions['A'].width = 30 # 设置行宽

    for i in range(length):
        ws['A' + str(i + 2)] = movie[i].title
        ws['B' + str(i + 2)] = movie[i].rank


    wb.save("top250.xlsx")

def main():
    host = "https://movie.douban.com/top250"
    res = get_page(host)
    depth = find_depth(res)

    for i in range(depth):
        url = host + '/?start=' + str(25*i)
        res = get_page(url)
        find_movies(res)

    rank_movies()

    # 把movies的内容写入到excle表中
    write_excel()

if __name__ == "__main__":
    main()


```
