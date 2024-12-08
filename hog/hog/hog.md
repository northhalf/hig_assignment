# 0. Question 0

这里是考察了`make_test_dice()`函数的用法，这个函数返回一个函数，之后调用这个返回的函数就是在原来传入的序列中不断遍历。`test_dice = make_test_dice(4, 1, 2)`，这里调用`test_dice()`返回值会4,1,2循环。

# 1. Problem 1

测试用例解锁是介绍了`roll_dice()`函数的用法，其中最重要的应该是，要确保调用了`dice()`函数`num_rolls`次，否则下次可能函数不在正确的位置了。

这个随机函数的实现还是很简单的，因为要确保`dice()`函数能够被调用`num_rolls`次，所以我选择直接把生成的值先放入一个列表，在从列表中依次检索，如果检索到1直接返回1，如果没有则累加值。最后返回累加的值。

```python
num = [dice() for i in range(num_rolls)]  # 随机数列表
num_sum = 0  # 记录数字和
for n in num:
    if n==1:  # 如果列表中出现1
        return 1  # 则返回1
    num_sum += n  # 累加随机生成的值
return num_sum
```

# 2. Problem 2

这个函数就是计算圆周率小数点第n位的值(0位为整数部分3)。题目提示用101位`pi`整数整除`pow(10,100-score)`进行截断就可以了，相当于保留score位小数部分：

```python
pi //= pow(10, 100-score)
```

# 3. Problem 3

函数为根据投多少骰子，对手有多少分数，随机生成函数来决定本轮获得的分数。分为两种情况来调用上面写的函数就可以：

```python
if num_rolls==0:  # 考虑不投骰子的特殊情况
    return free_bacon(opponent_score)
return roll_dice(num_rolls, dice)  # 正常投骰子的返回值
```

# 4. Problem 4

- [a](#a)
- [b](#b)

## a

就是求最大公约数，再判断是否大于或等于10。**不过**这里需要考虑分数为0的特殊情况，之后再用辗转相除求出最大公约数加上判断就好：

```python
if player_score==0 or opponent_score==0:  # 考虑分数为0的特殊情况
    return False
a = player_score  # 只是方便编写
b = opponent_score
while b:  # 辗转相除,最后a为最大公约数
    t = a%b
    a = b
    b = t
return True if a>=10 else False  # a>=10返回True否则False
```

## b

就是判断是否选手分数比对手低而且没到3分。直接`if`判断就行了：

```python
return True if -3<(player_score-opponent_score)<0 else False
```

# 5. Problem 5

完成游戏的一个主体。这里不是很明白`who`的意义，就是先是玩家1开始，然后玩家2，不断重复，直到有人达到目标分数即可，中间要注意字啊玩家1完成之后检查**是否已经达到分数**。并且在中间判断是否可以继续投骰子的时候也要判断是否在其过程中达到了分数，也就是时刻要注意选手分数的变化：

```python
while score0<goal and score1<goal:  # 直到有人达到目标分数才结束s
    if not who:  # 选手1
        # 选手1投一次骰子后的得分
        score0 += take_turn(strategy0(score0, score1), score1, dice)
        while (pig_pass(score0, score1) or swine_align(score0, score1)) \
                    and score0 <= goal:  # 判断是否可以再次投骰子
            score0 += take_turn(strategy0(score0, score1), score1, dice)
    else:  # 选手2
        # 选手2投一次骰子后的得分
        score1 += take_turn(strategy1(score1, score0), score0, dice)  
        while pig_pass(score1, score0) or swine_align(score1, score0) and score1 <= goal:  # 判断是否可以再次投骰子
            score1 += take_turn(strategy1(score1, score0), score0, dice)
    who = other(who)  # 更新who的值
```

# 6. Problem 6

这里要注意`say()`函数的调用，需要在每人投骰子之后就要调用，而不是两个人都投过之后才使用，每投一次调用一次。这里需要注意`say()`函数在每次调用之后要进行更新。将上面代码略加修改就好了：

```python
while score0<goal and score1<goal:  # 直到有人达到目标分数才结束s
    if not who:  # 选手1
        # 选手1投一次骰子后的得分
        score0 += take_turn(strategy0(score0, score1), score1, dice)  
        say = say(score0, score1)
        while (pig_pass(score0, score1) or swine_align(score0, score1)) \
                    and score0 <= goal:  # 判断是否可以再次投骰子
            score0 += take_turn(strategy0(score0, score1), score1, dice)
            say = say(score0, score1)
    else:  # 选手2
        # 选手2投一次骰子后的得分
        score1 += take_turn(strategy1(score1, score0), score0, dice)  
        say = say(score0, score1)
        while pig_pass(score1, score0) or swine_align(score1, score0) and score1 <= goal:  # 判断是否可以再次投骰子
            score1 += take_turn(strategy1(score1, score0), score0, dice)
            say = say(score0, score1)
    who = other(who)  # 更新who的值
```

# 7. Problem 7

这题需要我写一个返回函数的函数，这里只是要注意对于`running_high`的值不要轻易更改，不然会出现意想不到的问题。思路倒是很简单，就是不断更新`say()`函数的定义，也就是在`say()`函数内部调用`announce_highest()`函数，让其返回`say`函数，再返回这个`say`函数：

```python
def say(score_0, score_1):
    if who:  # 要求为选手1
        if score_1-last_score > running_high:  # 新的分数上升记录
            print(f"{score_1-last_score} point(s)! The most yet for Player 1")
            return announce_highest(who, score_1, score_1-last_score)
        return announce_highest(who, score_1, running_high)
    else:  # 选手0
        if score_0-last_score > running_high:  # 新的分数上升记录
            print(f"{score_0-last_score} point(s)! The most yet for Player 0")
            return announce_highest(who, score_0, score_0-last_score)
        return announce_highest(who, score_1, running_high)
return say
```

# 8. Problem 8

这题总而言之就是要计算所指定的骰子函数调用指定次数之后的平均值。因为考虑到骰子函数的参数数量和类型不同，所以要使用不定参数。代码倒是很容易写：

```python
def average(*arg):
    # 投骰子目标次数生成的点数
    num = (original_function(*arg) for i in range(trials_count))
    points_sum = sum(num)  # 计算总点数
    return points_sum/trials_count
return average
```

# 9. Problem 9

这里就是计算每局投多少次骰子使得每局平均分最高，有`trials_count`局。那么将1-10种可能性得到的每局平均分记录下来，再找到其中最高的就行了。

```python
average = []  # 存储投骰子次数从1到10的每局分数平均值
for i in range(1, 11):
    score = (roll_dice(i, dice) for n in range(trials_count))  # 每局分数的生成器
    average.append(sum(score) / (trials_count))
return average.index(max(average))+1
```

# 10. Problem 10

这题就是调用`free_bacon()`函数看有没有达到截断点，达到就返回0，否则返回`num_rolls`。

```python
return 0 if free_bacon(opponent_score)>=cutoff else num_rolls
```

# 11. Problem 11

就是判断不投骰子是否能够满足能够再次投骰子或者达到截断点

```python
# 依次检测是否满足截断点，猪之传递或豕之同步
return 0 if bacon_strategy(score, opponent_score, cutoff, num_rolls)==0\
            or pig_pass(score+free_bacon(opponent_score), opponent_score) \
            or swine_align(score+free_bacon(opponent_score), opponent_score)\
        else num_rolls
```

# 12. Problem 12

编写最后的策略，这里没有骰子的类型，只能依靠双方分数实行策略。那么首先从`extra_turn_strategy()`开始检测(截断点为18,假设为6面骰子)，如果0次比较好那就是0次。如果不是，那么检测是否投0次可以胜利(达到100分)。如果都不行，那么投4次，因为在6面骰子的情况下，投4次能有接近50%的概率不是1。

```python
if not extra_turn_strategy(score, opponent_score,18):
    return 0
if free_bacon(opponent_score)+score >= 100:  # 检查得分在不投的情况下是否达到100
    return 0
return 4
```

