**1. 问题：写程序来递归统计某个文件夹下的所有文件（多层目录）**

思路：
- 访问该目录，遍历其中的文件/文件夹，如果是文件就添加到最终结果，如果是文件夹就递归调用访问目录的函数
- 限制递归调用深度，防止深度过深导致堆栈溢出


实现：
>这里用C在处理返回数据时比较繁琐，此处使用GoLang实现


```
package main

import (
	"fmt"
	"io/ioutil"
)

const MAXDEPTH uint = 1000

func main() {
	dir := "D:\\workspace\\src"
	files, err := GetAllFiles(dir, 1)
	if err != nil {
		fmt.Println("get all files failed", err)
	}
	for _, file := range files {
		fmt.Println(file)
	}
}

func GetAllFiles(dir string, depth uint) ([]string, error) {
	if depth > MAXDEPTH {
		return []string{}, nil
	}
	_, err := os.Stat(dir)
	if os.IsNotExist(err) {
		return []string{}, err
	}
	depth++
	filesAll := []string{}

	files, err := ioutil.ReadDir(dir)
	if err != nil {
		fmt.Println("read dir failed", dir, err)
	}

	for _, file := range files {
		fullName := dir + "\\" + file.Name()
		if file.IsDir() {
			temp, err := GetAllFiles(fullName, depth)
			if err != nil {
				fmt.Println("read dir failed", fullName, err)
			}
			filesAll = append(filesAll, temp...)
		} else {
			filesAll = append(filesAll, fullName)
		}
	}
	return filesAll, nil
}

```

运行结果示例：
>D:\workspace\src\GetAllFiles\GetAllFiles.exe
>D:\workspace\src\GetAllFiles\main.go
>D:\workspace\src\GitHub\beginner\README.md
>D:\workspace\src\GitHub\beginner\demo_week04\note.md
>D:\workspace\src\GitHub\beginner\demo_week05\微服务可用性设计总结.md
>D:\workspace\src\GitHub\beginner\demo_week2\main.go
>D:\workspace\src\GitHub\beginner\demo_week3\main.go
>D:\workspace\src\GitHub\beginner\demo_week3\note.md
>D:\workspace\src\proto_xml_gen\BNFsample1.txt
>D:\workspace\src\proto_xml_gen\BNTtoXML.go
>D:\workspace\src\proto_xml_gen\generateXML.go
>D:\workspace\src\proto_xml_gen\main.go


测试情况：

* 基本功能
* 边界情况，比如：
    正常情况：
    - 文件夹下没有文件也没有目录
    - 没有文件有目录
    - 目录层级过多等（可以用降低最大深度来辅助测试）
    
    异常情况：
    - 指定目录不存在等

测试举例：

```
package main

import (
	"os"
	"testing"
)

func Test_GetAllFilesNull(t *testing.T) {
	temp, _ := GetAllFiles("D:\\workspace\\null", 1)
	if len(temp) == 0 {
		t.Log("read null dir pass")
	} else {
		t.Log(temp)
		t.Error("read null dir failed")
	}
}

func Test_GetAllFilesNotExist(t *testing.T) {
	_, err := GetAllFiles("D:\\workspace\\temp", 1)
	if os.IsNotExist(err) {
		t.Log("read not exist dir pass")
	} else {
		t.Error("read not exist failed")
	}
}

```


**2. 问题：最大匹配无重复子串**

思路：
- 用一个整型数组来存放最近一次某字符出现的位置，数组元素初始化为-1
- 初始化最长无重复子串的开始位置和结束位置
- 从字符串起始处开始遍历每一个字符
- 如果该字符没有出现过（用s[i] - ‘ ’作为数组索引），就将其加入出现过的数组中，值为其出现的位置
- 如果该字符出现过，先判断当前位置-1到子串开始位置的长度是否大于原本的最大长度，大于的话更新目前找到的最长无重复子串的长度，然后重新将子串的左、右边界置为当前位置，将当前位置之前的元素从出现过的数组里剔除，将当前位置的字符加入出现过的数组，直到到达字符串结尾
- 最后一个字符跟目前左右边界中的子串无重复时，再次更新最大无重复子串的值

实现：
>C语言

```
#include <string.h>
#include <stdio.h>

int LongestSubstring(char * s){
    int exist[255] = {0};
    int left = 0;
    int right = 0;
    int maxLen = 0;
    int tempLeft = 0;
    int j = 0;
    memset(exist, -1, sizeof(exist));
    for (right = 0; right < strlen(s); right++) {
        if (exist[s[right]-' '] != -1) {
            if (maxLen < (right - left)) {
                maxLen = right - left;
            }
            tempLeft = left;
            left = exist[s[right]-' '] + 1;
            for (j = tempLeft; j < left; j++) {
                if (s[j] != s[left]) {
                    exist[s[j]-' '] = -1;
                }
            }
        }
        exist[s[right]-' '] = right;
    }

    if (right == strlen(s) && maxLen < (right - left)) {
        maxLen = right - left;
    } 

    return maxLen;
}

void testLongestSubstring(char * s, int len) {
    if (LongestSubstring(s) == len) {
        printf("test %s pass\n", s);
    } else {
        printf("test %s failed\n", s);
    }
}


int main() {
    testLongestSubstring("abcabcbb", 3);
    //测试全重复的字符串
    testLongestSubstring("bbbbb", 1);
    testLongestSubstring("pwwkew", 3);
    //测试空字符串
    testLongestSubstring("", 0);
    //测试只有一个空格的字符串
    testLongestSubstring(" ", 1);
    //测试无重复的字符串
    testLongestSubstring("au", 2);
    //测试重复字符在一开始的字符串
    testLongestSubstring("aab", 2);
}

```

测试结果：

>test abcabcbb pass
>
>test bbbbb pass
>
>test pwwkew pass
>
>test  pass
>
>test   pass
>
>test au pass
>
>test aab pass




