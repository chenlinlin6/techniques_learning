今天开始从简单的开始总结一些简单的算法题。  
我按照leetcode interview的题从简单到困难排序，依次选取题目进行解析。
## 倒置文本
![倒置文本](http://upload-images.jianshu.io/upload_images/2338511-975ce33172e66d17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这道题是将文本倒置，比如把'hello'倒置以后就是'olleh'。
### 解法一 直接用Python列表内置的倒序功能
这道题最简单的办法就是
```
class Solution(object):
	def reverse_string(s):
		return s[::-1]
```
> 这里有stackflow的[解释](https://stackoverflow.com/questions/21617586/reverse-string-string-1-works-but-string0-1-and-others-dont)。
> 也就是Slice notation "[a:b:c]" means "count in increments of c starting at a inclusive, up to b exclusive". If c is negative you count backwards, if omitted it is 1. If a is omitted then you start as far as possible in the direction you're counting from (so that's the start if c is positive and the end if negative). a和b分别是开头和结尾的索引位置，c正负代表方向。  

### 解法二 分而治之  
总体思路是以中间的index为分割点，按照尾首这样的链接方式连接，最终退出循环的方式就是只剩下一个元素。  
```
class Solution(object):
	def reverse_string(s):
		l=len(s)
		if l==1:
			return s
		return reverse_string(s[l/2:])+reverse_string(s[:l/2])
```
>注意两点，一个是设定退出循环的条件，当只有一个的时候就要退出，第二个就是每一层要做的事情（在这里是要首尾调换）

## Fizz Buzz
[出处](https://leetcode.com/problems/fizz-buzz/description/)
![](http://upload-images.jianshu.io/upload_images/2338511-4ab77200423644e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

问题描述：这道题目的是根据输入的数字做一个遍历，对于3和5的倍数分别做不同的输出。简单的思路是直接用条件语句，但注意到题干的特点是当为3和5共同的倍数的时候输出的就是它们各自的输出的连接，那么实际上应该要对除以3和5的余数是否为0先做个判断，然后再看是否连接，这样似乎更加有条理一些。  
```
class Solutions(object):
	def fizzbuzz(n):
		return ['Fizz'*(not i%3) + 'Buzz'*(not i%5) or str(i) for i in range(n+1)]
```
![](http://upload-images.jianshu.io/upload_images/2338511-06fba44384fd7375.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Single Number
[出处](https://leetcode.com/problems/single-number/description/)
[图片上传失败...(image-36be30-1520155685284)]
问题描述：一个由数字组成的序列，其中除了一个数字只出现了一遍，所有数字都出现了2次，请找出没有重复的那个数字。  
这道题目的是检查重复的数字，其实可以用一个比较tricky的办法来解决，就是XOR（异或）。
[图片上传失败...(image-c5a53b-1520155685284)]
简单来说，异或检查是否是相同的，对于相同的数字就会返回0，不同的就会返回它们的和，又叫半加算法。并且运算的结果与顺序是无关的（可以理解，因为只是要检查是否是相同的）  
![](http://upload-images.jianshu.io/upload_images/2338511-de189e55842a5f4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> A better explanation why this technique works-
Let’s say we have an array - [2,1,4,5,2,4,1].
What we are doing is essentially this-
=> 0 ^ 2 ^ 1 ^ 4 ^ 5 ^ 2 ^ 4 ^ 1
=> 0^ 2^2 ^ 1^1 ^ 4^4 ^5 (Rearranging, taking same numbers together)
=> 0 ^ 0 ^ 0 ^ 0 ^ 5
=> 0 ^ 5
=> 5 :)

因为最后要返回那个值，所以我们应该初始值为0.
```
class Solutions(object):
	def find_single_number(self, digit_list):
		result=0
		for num in digit_list:
			result ^= num
		return result
```
![屏幕快照 2018-03-04 下午5.23.16.png](http://upload-images.jianshu.io/upload_images/2338511-4d1cf837d9193bc8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

