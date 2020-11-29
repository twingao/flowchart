# yaml疑难语法说明

安装PyYAML模块。

    yum install python-yaml

可以将yaml文件转为json格式输出。indent=2为缩进2空格，ensure_ascii=False可以输出汉字。

    python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=2, ensure_ascii=False)' < test.yaml

yaml字符串多行。

    vi test.yaml
    str: 这是一段
      多行
      字符串

    python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=2, ensure_ascii=False)' < test.yaml
    {
      "str": "这是一段 多行 字符串"
    }

换行符会被转义为空格，每一行前面都需要有空格，空格数不限。

    vi test.yaml
    str: 这是一段
    多行
    字符串

    python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=2, ensure_ascii=False)' < test.yaml
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/usr/lib64/python2.7/site-packages/yaml/__init__.py", line 71, in load
        return loader.get_single_data()
      File "/usr/lib64/python2.7/site-packages/yaml/constructor.py", line 37, in get_single_data
        node = self.get_single_node()
    ......

以上是格式错误的。

    vi test.yaml
    str: 这是一段
                    多行
                                             字符串
    
    python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=2, ensure_ascii=False)' < test.yaml
    {
      "str": "这是一段 多行 字符串"
    }

或

    vi test.yaml
    str:
     这是一段
     多行
     字符串
    
    python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=2, ensure_ascii=False)' < test.yaml
    {
      "str": "这是一段 多行 字符串"
    }

或

    vi test.yaml
    str:
      这是一段
      
      多行
      字符串

    python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=2, ensure_ascii=False)' < test.yaml
    {
      "str": "这是一段\n多行 字符串"
    }

空行表示一个新行，不会将换行符删除。

可以通过|保留换行符。注意"字符串"后面并没有换行符，但最后会加上换行符。

    vi test.yaml
    str: |
      这是一段
      多行
      字符串
    
    python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=2, ensure_ascii=False)' < test.yaml
    {
      "str": "这是一段\n多行\n字符串\n"
    }

或

    vi test.yaml
    str: |
      这是一段
    
      多行
      字符串
    
    python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=2, ensure_ascii=False)' < test.yaml
    {
      "str": "这是一段\n\n多行\n字符串\n"
    }

或

    vi test.yaml
    str: |
      这是一段
      多行
      字符串
    
    str2: 字符串2

    python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=2, ensure_ascii=False)' < test.yaml
    {
      "str2": "字符串2",
      "str": "这是一段\n多行\n字符串\n"
    }

注意我们在"字符串"后面增加一个空行，空行被忽略。

可以通过>折叠字符串，将多行中间的换行符转换为空格，但保留最后一个换行符。

    vi test.yaml
    str: >
      这是一段
      多行
      字符串

    python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=2, ensure_ascii=False)' < test.yaml
    {
      "str": "这是一段 多行 字符串\n"
    }

或

    vi test.yaml
    str: >
      这是一段
    
      多行
      字符串

    python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=2, ensure_ascii=False)' < test.yaml
    {
      "str": "这是一段\n多行 字符串\n"
    }

空行可以保留。

可以通过|+保留末尾的换行符。以下和|没有区别。

    vi test.yaml
    str: |+
      这是一段
      多行
      字符串

    python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=2, ensure_ascii=False)' < test.yaml
    {
      "str": "这是一段\n多行\n字符串\n"
    }

或

    vi test.yaml
    str: |+
      这是一段
      多行
      字符串
    
    str2: 字符串2

    python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=2, ensure_ascii=False)' < test.yaml
    {
      "str2": "字符串2",
      "str": "这是一段\n多行\n字符串\n\n"
    }

注意以上"字符串"后面有一个空行。

或

    vi test.yaml
    str: |+
      这是一段
      多行
      字符串
    

    python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=2, ensure_ascii=False)' < test.yaml
    {
      "str": "这是一段\n多行\n字符串\n\n"
    }

注意以上"字符串"后面也有一个空行。

可以通过|-删除末尾的换行符。

    vi test.yaml
    str: |-
      这是一段
      多行
      字符串

    python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=2, ensure_ascii=False)' < test.yaml
    {
      "str": "这是一段\n多行\n字符串"
    }

或

    vi test.yaml
    str: |-
      这是一段
      多行
      字符串
    
    str2: 字符串2

    python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=2, ensure_ascii=False)' < test.yaml
    {
      "str2": "字符串2",
      "str": "这是一段\n多行\n字符串"
    }

注意以上"字符串"后面有一个空行。

    vi test.yaml
    str: |-
      这是一段
      多行
      字符串
    

    python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=2, ensure_ascii=False)' < test.yaml
    {
      "str": "这是一段\n多行\n字符串"
    }

注意以上"字符串"后面也有一个空行。

