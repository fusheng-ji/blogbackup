# linux命令行单词翻译工具——fy

## 安装

```bash
# 首先安装ruby依赖
sudo apt-get install ruby
# 安装fy
sudo gem install fy
```

## 单个单词翻译

```bash
# fy 单词 即可
$ fy love
/var/lib/gems/2.7.0/gems/fy-1.0.1/lib/fy/fanyi.rb:17: warning: URI.escape is obsolete
 love [ lʌv ]

 - n. 爱；爱情；喜好；（昵称）亲爱的；爱你的；心爱的人；钟爱之物；零分
 - v. 爱恋（某人）；关爱；喜欢（某物或某事）；忠于
 - n. (Love) （英、菲、瑞、美）洛夫（人名）

 1. love
    爱, 爱情, 爱心, 恋爱
 2. First love
    初恋, 初恋这件小事, 魔女的条件主题曲
 3. love Letter
    情书, 日本版, 爱的情书, 爱情信
# 翻译中文
$ fy 这
/var/lib/gems/2.7.0/gems/fy-1.0.1/lib/fy/fanyi.rb:17: warning: URI.escape is obsolete
 这 [ zhèi,zhè ]

 - this
 - it
 - these
 - now

 1. 初恋这件小事
    First Love, A Little Thing Called Love, First love this little thing, The Crazy Little Thing called Love
 2. 这里
    Here, over here, soulja, HR
 3. 这些
    These, the, thesepron, No, not those

```

## 多个单词翻译

```bash
# fy 回车 即可开启多个单词连续翻译模式
# 使用ctrl+c退出该模式
$ fy
> words
/var/lib/gems/2.7.0/gems/fy-1.0.1/lib/fy/fanyi.rb:17: warning: URI.escape is obsolete
 words [ wɜːdz ]

 - n. [计] 字（word的复数）；话语；言语
 - v. 用言语表达（word的三单形式）

 1. words
    言辞, 字数, 字样, 字眼
 2. in other words
    换句话说, 换言之, 也就是说, 换句话讲
 3. Stop words
    停用词, 禁用词, 连接词, 停止词

> even
/var/lib/gems/2.7.0/gems/fy-1.0.1/lib/fy/fanyi.rb:17: warning: URI.escape is obsolete
 even [ ˈiːvn ]

 - adj. 平坦的，水平的；平静的，平和的；平齐的，挨着的；无变化的，平稳的；势均力敌的，不相上下的；双数的，偶数的；平等的，公平的；不变的，恒定的；（某事发生的概率）对半的，各半的；扯平的，互不相欠的；匀称的，同样大小的；正好的，不多不少的
 - adv. 甚至，即使；更加，愈加；其实，实际上
 - v. 使相等，使平均；变平坦；使变平，使平整
 - n. <文>傍晚，晚上
 - 【名】 （Even）（挪、美）埃旺（人名）

 1. even
    甚至, 偶数, 哪怕, 连
 2. even if
    即使, 虽然, 尽管, 纵然
 3. even though
    即使, 尽管, 纵然, 虽然
```

## 翻译句子

```bash
$ fy i love linux
/var/lib/gems/2.7.0/gems/fy-1.0.1/lib/fy/fanyi.rb:17: warning: URI.escape is obsolete
 i love linux

  我喜欢linux

 1. i love linux
    自我介绍

$ fy 有朋自远方来
/var/lib/gems/2.7.0/gems/fy-1.0.1/lib/fy/fanyi.rb:17: warning: URI.escape is obsolete
 有朋自远方来

 - It is always a pleasure to greet a friend from afar

 1. 有朋自远方来
    Friends from afar, A friend coming from afar, Old Friend from Far Away
 2. 有朋自远方来不亦乐乎
    Welcome my friends
 3. 有朋自远方来访喜相迎
    Oliver Hayakawa

```

