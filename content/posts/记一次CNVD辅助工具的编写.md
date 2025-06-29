---
title: "记一次CNVD辅助工具的编写"
date: 2024-06-04T08:42:02+08:00
draft: false
categories:
- 学习记录
tags:
- Pyhton
- 工具
---

# 思路

很久没写过Python代码，最近在挖CNVD也不知道该搜什么关键字，遂产生了一个想法：提取爱企查企业软件著作权信息列表的软件名称作为fofa搜索的关键字，返回fofa搜索到的总数，低于10的pass掉，再手工去搜索系统挖洞。

# 实现

1. 爱企查搜索每个公司会有一个pid，类似于企业在爱企查的身份证号，查询其他信息离不开pid，先获取pid：

```python
def get_pid(company, header):
    company_name_urlencoded = quote(company)
    url = "https://aiqicha.baidu.com/s?q=" + company_name_urlencoded
    timout = get_random_timeout()
    response = requests.get(url=url, headers=header, timeout=timout)
    if response.status_code == 200:
        text = response.text
        pidmatch = re.search(r"\d{14}", text)
        pid = pidmatch.group()
        # print(pid)
        return pid
    else:
        print("页面请求失败1：", response.status_code)
        return None
```

2. 软件著作权信息默认只显示10行，大于10行就要进行翻页，那么需要翻几页呢？这里需要获取pageCount数值：

```python
def get_pagecount(pid, header):
    url = "https://aiqicha.baidu.com/cs/copyrightAjax?pid=" + pid
    timout = get_random_timeout()
    response = requests.get(url=url, headers=header, timeout=timout)
    if response.status_code == 200:
        text = json.loads(response.text)
        pagecount = text["data"]["pageCount"]
        return pagecount
    else:
        print("页面请求失败2：", response.status_code)
        return None
```

3. 至此就可以通过pid指定搜索的企业，通过pageCount对软件著作权信息进行翻页，下面就爬取所有软件名保存到文件里：

```python
def get_softwarename(pid, pagecount, header):
    with open("softwareName.txt", "w", encoding="utf-8") as softName_file:
        for i in range(1, pagecount):
            url = "https://aiqicha.baidu.com/cs/copyrightAjax?p=" + str(i) + "&pid=" + pid
            timout = get_random_timeout()
            response = requests.get(url=url, headers=header, timeout=timout)
            if response.status_code == 200:
                text = response.text
                unique_software_names = set()
                matches = re.findall(r'"softwareName":"([^"]*)"', text)
                for match in matches:
                    if match not in unique_software_names:
                        unique_software_names.add(match)
                        # print(match)
                        softName_file.writelines(match + "\n")

            else:
                print("页面请求失败2：", response.status_code)
                return None
```

4. 最后通过读取文件名到fofa进行搜索，并获取搜索到的数量：

```python
def get_fofa_result():
    with open("softwareName.txt", "r", encoding="utf-8") as fofa_file:
        softwarename = fofa_file.readlines()
        # print(softwarename)
        for i in range(0, len(softwarename) - 1):
            statement = "title=\"" + softwarename[i] + "\" || body=\"" + softwarename[i] + "\""
            data = {
                "key": "[这里填fofa-key]",
                "qbase64": base64.b64encode(statement.encode("UTF-8"))
            }
            timout = get_random_timeout()
            req = requests.get(url='https://fofa.info/api/v1/search/all', verify=True, params=data, timeout=timout)
            if req.status_code != 200:
                print('fofa 请求失败，请检查配置')
                return False

            data = req.json()
            if data.get("error", True) is not False:
                print('fofa 请求失败，请检查配置')
                return False

            text = json.loads(req.text)
            num = text["size"]
            with open("targetinfo.txt", "a", encoding="utf-8") as target_file:
                if num < 10:
                    pass
                else:
                    content = softwarename[i].strip() + "————————————" + str(num) + "个"
                    print(content)
                    target_file.writelines(content + "\n")
```

这次功能已经完成，来看看效果怎么样：

![](https://s2.loli.net/2024/06/11/i27nMJHG364QLD9.png)

# 缺点

爱企查存在安全验证，用几次就得到爱企查验证一下，水平有限不知道怎么过验证；

未考虑并发。
