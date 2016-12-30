**author: fzu lkl**

## 一个使用LR(1)分析法并采用语法制导的简单编译器前端
### 一、词法分析器

词法分析部分参考龙书附录A的词法分析器

**注：float改为double**

**词法分析器中指定从src目录中的test.txt文件读入测试代码**

### 二、语法分析器

#### 设计思路
**语法分析器采用LR(1)分析法**

1. 对给定的原始文法进行处理，非终结符以单个大写字母表示，终结符以小写字母或单个字符表示。

2. 产生式左部与右部以“：”分隔，对于具有相同左部的产生式以“|”分隔，不同产生式间以“，”分隔，如：“E：E+T|a，T：c”。

3. 文法类Grammer，用于对输入的产生式进行处理保存所有产生式，并求出终结符集合、非终结符集合、可推出空的非终结符结合、所有非终结符的first集。

4. Item类表示项，Item类的每个对象保存有该项对应的产生式左部、右部、期待读入的字符位置、前看符号集合。该类中有一个FirstS()用于求项集闭包时由该项推出的项的前看符合集合。

5. SetOfItems类表示项集，该类的对象中保存当前项集状态标号和项集中的所有项。有一个closure方法用于求该项集的闭包，一个reduce(char x)方法用于在构建分析表的时候，判断输入为x时是否可以归约和按哪个产生式归约。一个goTo(char x)方法用于构建分析表时判断移进x将由该项集(状态)转移到哪个项集（状态）。

6. 预测分析表用HashMap<String,String>存储，key为当前状态+移进的token，value为“ri”表示按第i条产生式归约，“si”移进并转移到状态i，“gi”表示移进并跳转到状态i。

7. Chart类用于驱动生成分析表，并保存有分析表和所有状态集。其中方法initParseChart(initSet)用于生成预测分析表，参数initSet为初始项集。

8. Parser类用于使用预测分析表进行语法分析，类中持有词法分析器对象和一个栈，栈用于保存移进的token和对应的状态。exchange(Token token)方法用于将token转换成对应的终结符。parse()方法用于进行语法分析，根据预测分析表判断对读入的字符要进行的动作。

#### 测试例子:  
**输入:**

```
    {
        int i;
        i = i + i;
    }
```

**输出:**

[0]	{	移入

[0, {, 1]	b	根据D:#归约

[0, {, 1, D, 4]	b	移入

[0, {, 1, D, 4, b, 6]	i	根据T:b归约

[0, {, 1, D, 4, T, 9]	i	移入

[0, {, 1, D, 4, T, 9, i, 20]	;	移入

[0, {, 1, D, 4, T, 9, i, 20, ;, 29]	i	根据A:Ti;归约

[0, {, 1, D, 4, A, 8]	i	根据D:DA归约

[0, {, 1, D, 4]	i	根据S:#归约

[0, {, 1, D, 4, S, 7]	i	移入

[0, {, 1, D, 4, S, 7, i, 12]	=	根据L:i归约

[0, {, 1, D, 4, S, 7, L, 19]	=	移入

[0, {, 1, D, 4, S, 7, L, 19, =, 28]	i	移入

[0, {, 1, D, 4, S, 7, L, 19, =, 28, i, 68]	+	根据L:i归约

[0, {, 1, D, 4, S, 7, L, 19, =, 28, L, 76]	+	根据F:L归约

[0, {, 1, D, 4, S, 7, L, 19, =, 28, F, 84]	+	根据U:F归约

[0, {, 1, D, 4, S, 7, L, 19, =, 28, U, 83]	+	根据I:U归约

[0, {, 1, D, 4, S, 7, L, 19, =, 28, I, 82]	+	根据H:I归约

[0, {, 1, D, 4, S, 7, L, 19, =, 28, H, 81]	+	移入

[0, {, 1, D, 4, S, 7, L, 19, =, 28, H, 81, +, 136]	i	移入

[0, {, 1, D, 4, S, 7, L, 19, =, 28, H, 81, +, 136, i, 68]	;	根据L:i归约

[0, {, 1, D, 4, S, 7, L, 19, =, 28, H, 81, +, 136, L, 76]	;	根据F:L归约

[0, {, 1, D, 4, S, 7, L, 19, =, 28, H, 81, +, 136, F, 84]	;	根据U:F归约

[0, {, 1, D, 4, S, 7, L, 19, =, 28, H, 81, +, 136, U, 83]	;	根据I:U归约

[0, {, 1, D, 4, S, 7, L, 19, =, 28, H, 81, +, 136, I, 188]	;	根据H:H+I归约

[0, {, 1, D, 4, S, 7, L, 19, =, 28, H, 81]	;	根据R:H归约

[0, {, 1, D, 4, S, 7, L, 19, =, 28, R, 80]	;	根据E:R归约

[0, {, 1, D, 4, S, 7, L, 19, =, 28, E, 79]	;	根据J:E归约

[0, {, 1, D, 4, S, 7, L, 19, =, 28, J, 78]	;	根据G:J归约

[0, {, 1, D, 4, S, 7, L, 19, =, 28, G, 77]	;	移入

[0, {, 1, D, 4, S, 7, L, 19, =, 28, G, 77, ;, 127]	}	根据C:L=G;归约

[0, {, 1, D, 4, S, 7, C, 18]	}	根据S:SC归约

[0, {, 1, D, 4, S, 7]	}	移入

[0, {, 1, D, 4, S, 7, }, 11]	$	根据B:{DS}归约

[0, B, 2]	$	根据P:B归约

[0, P]	$	acc

### 语义分析
#### 尚未完成
