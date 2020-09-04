---
title: 基于阿里云api的动态域名解析服务
tags: 'C#'
categories: 小工具
abbrlink: 62468
date: 2018-06-12 12:00:34
---
基于阿里云解析服务API的DDNS服务。将本机IP更新至指定域名的DNS A记录，可以达到花生壳动态域名解析的效果。

<!-- more -->
[源码地址](https://github.com/AlanLang/aliyun-ddns-server)

## 使用方法
1. 在阿里云申请一个域名，将此域名添加一个子域（如www），并设置为A类型记录，IP地址随便填写一个（程序会自动修改）
2. 到阿里云域名控制台申请AccessId Key和Secrect
3. Clone本项目代码到本机，使用VS2013或更高版本编译 (或直接下载release)
4. 将生成的debug目录拷贝到服务器上，修改config.yml文件，然后用管理员权限运行install.bat

## 配置说明
`config.yml`

``` ymal
# 检测外网ip地址的网站
IPUrl: http://2018.ip138.com/ic.asp
# 阿里云接口地址
APIUrl: http://alidns.aliyuncs.com
DomainName: 域名 例如 google.com
FirstName: 前缀，例如www,空则为 @
AccessKey: Access Id Key
AccessKeySecret: Access Id Secret
```

## 环境
使用 vs2015+C# 开发 .NET 4.0

## 关键代码
首先下载并引用阿里云sdk
``` C
/// <summary>
/// 域名解析帮助类
/// </summary>
public class CDomainHelper
{
    DefaultAliyunClient aliyunClient;
    public CDomainHelper()
    {
        aliyunClient = new DefaultAliyunClient(CGlobalConfig.APIUrl, CGlobalConfig.AccessKey, CGlobalConfig.AccessKeySecret);
    }
    /// <summary>
    /// 获取当前的解析值
    /// </summary>
    /// <returns></returns>
    public Record DescribeDomains()
    {

        var req = new DescribeDomainRecordsRequest() { DomainName = CGlobalConfig.DomainName };
        var response = aliyunClient.Execute(req);

        var updateRecord = response.DomainRecords.FirstOrDefault(rec => rec.RR == CGlobalConfig.FirstName && rec.Type == "A");
        return updateRecord;

    }
    /// <summary>
    /// 更新域名解析值
    /// </summary>
    /// <param name="ipaddr">新的ip地址</param>
    /// <param name="recordId">解析条目的主键</param>
    public void UpdateDomainRecords(string ipaddr,string recordId)
    {
        var changeValueRequest = new UpdateDomainRecordRequest()
        {
            RecordId = recordId,
            Value = ipaddr,
            Type = "A",
            RR = CGlobalConfig.FirstName
        };
        aliyunClient.Execute(changeValueRequest);
    }
    /// <summary>
    /// 获取外网ip地址
    /// </summary>
    /// <returns></returns>
    public string GetIpAddr()
    {
        HttpWebRequest request = HttpWebRequest.Create(CGlobalConfig.IPUrl) as HttpWebRequest;
        using (HttpWebResponse response = request.GetResponse() as HttpWebResponse)
        {
            StreamReader reader = new StreamReader(response.GetResponseStream(), Encoding.GetEncoding("GB2312"));
            string str = reader.ReadToEnd();
            reader.Close();
            reader.Dispose();
            int start = str.IndexOf("[");
            int end = str.IndexOf("]");
            if (start > -1 && end > -1 && end > start)
            {
                string ip = str.Substring(start + 1, end - start - 1);
                return ip;
            }
            return "";
        }
    }
}
```