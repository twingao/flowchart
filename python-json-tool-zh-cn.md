# python -m json.tool中文乱码问题

我们在bash中可以通过python -m json.tool将json字符串格式化，以易于阅读的缩进方式输出到控制台上。如：

    echo '{"key1":"value1","key2":{"key3":"value3","key4":"value4"}}' | python -m json.tool
    {
        "key1": "value1",
        "key2": {
            "key3": "value3",
            "key4": "value4"
        }
    }

如果json字符串中包含了中文字符，python会以unicode格式显示，不易阅读。

    echo '{"key1":"value1","key2":{"key3":"张三","key4":"李四"}}' | python -m json.tool
    {
        "key1": "value1",
        "key2": {
            "key3": "\u5f20\u4e09",
            "key4": "\u674e\u56db"
        }
    }

解决这个问题的办法是修改json.tool程序，该程序存在于python系统库安装路径下的json/tool.py。

    find / -name tool.py
    /usr/lib64/python2.7/json/tool.py

json.dump方法中增加参数ensure_ascii=False，即让json.tool程序不强行保证json的内容都转义为ascii编码，中文原样输出即可。

    vi /usr/lib64/python2.7/json/tool.py
            json.dump(obj, outfile, sort_keys=True,
                      indent=4, separators=(',', ': '))
    # 改为
            json.dump(obj, outfile, sort_keys=True,
                      indent=4, ensure_ascii=False, separators=(',', ': '))

现在可以正常输出中文字符了。

    echo '{"key1":"value1","key2":{"key3":"张三","key4":"李四"}}' | python -m json.tool
    {
        "key1": "value1",
        "key2": {
            "key3": "张三",
            "key4": "李四"
        }
    }
