# profanity

波场（TRON）靓号生成器，利用 `gpu` 进行加速。

<img width="100%" src="https://github.com/GG4mida/profanity-tron/blob/main/screenshot/demo.png?raw=true"/>

> 该程序仅用于学习交流，请勿用于非法用途。
> 本程序仅在该仓库发布，请勿下载运行其它来路不明的版本，由此造成的一切损失，请自行承担。

## 说明

- 本软件基于以太坊靓号生成器：[profanity](https://github.com/johguse/profanity) 修改而来，同时已修复原程序私钥可爆破的问题。可参考下方 `安全` 章节说明。
- 本软件目前仅提供可执行程序，生成的地址请务必进行多签后再使用。多签后的地址，可保证 `100%` 安全，具体如何多签，可参考下方 `安全` 章节说明。
- 如果不想多签，又有安全方面的顾虑，请勿下载使用。不然徒增烦恼 :>
- 需要源码定制的可电报联系作者。

## 运行

```plaintext
Usage: profanity.exe [OPTIONS]

  Help:
    --help              Show help information

  Modes with arguments:
    --matching          Matching address list file, please refer to matching.txt

  Matching configuration:
    --prefix-count      Minimum number of prefix matches, default 0
    --suffix-count      Minimum number of suffix matches, default 6
    --quit-count        Exit the program when the generated number is greater than, default 0

  Device control:
    --skip              Skip device given by index

  Output control:
    --output            The file to output the results to
    --post              The url to post the results to

Examples:

  ./profanity --matching profanity.txt
  ./profanity --matching profanity.txt --skip 1
  ./profanity --matching profanity.txt --output result.txt
  ./profanity --matching profanity.txt --post http://127.0.0.1:7001/api
  ./profanity --matching profanity.txt --prefix-count 1 --suffix-count 8
  ./profanity --matching profanity.txt --prefix-count 1 --suffix-count 10 --quit-count 1

About:

  Profanity is a vanity address generator for Tron: https://tron.network
  The software is modified based on ethereum profanity: https://github.com/johguse/profanity
  Author: telegram -> @jackslowfak

Fbi Warning:

  Before using a generated vanigity address, always verify that it matches the printed private key.
  And always multi-sign the address to ensure account security.
```

### 参数说明

- --help   查看帮助说明
- --matching   固定写法，后面跟上匹配规则文件
- --prefix-count   最少匹配前缀位数，默认 0。比如你可以配置为 8，那就匹配 8 个 T 的地址。
- --suffix-count   最少匹配后缀位数，默认 6。比如你可以配置为 10，那就匹配 10 位的后缀（10位其实挺难的，估计要跑到天荒地老 :<）
- --quit-count   生成的地址达到指定的数量，即退出程序。比如你就想匹配一个地址，那就配置为 1。系统默认退出数量为 120。
- --output   将生成的地址输出到文件（追加）。一行一个，格式如：privatekey,address
- --post   将生成的地址，发送到（POST）指定的 url，每生成一条就会发送一次。数据格式为：privatekey=xx&address=yy。这个配置主要便于其它系统的集成。
- --skip   跳过指定索引的 gpu 设备

### 匹配规则

关于地址匹配规则，目前支持两种写法，可参考内置 `profanity.txt`。举个例子：

```plaintext
TTTTTTTTTTZZZZZZZZZZ
TUqEg3dzVEJNQSVW2HY98z5X8SBdhmao8D
```

上面这两条匹配规则：
- 第一条，是匹配以字母 `Z` 结尾的靓号。
- 第二条，表示是匹配这条地址的前后 `10` 位，实际运行的时候，会自动修正为：TUqEg3dzVE8SBdhmao8D。

有了匹配规则，再结合 `prefix-count`（最少匹配前缀数量） & `suffix-count`（最少匹配后缀数量），即可实现靓号地址生成。

### Windows 

直接运行 `start.bat` 即可。

> 请自行编辑 `start.bat` 配置运行参数。

### Mac

`mac` 环境一般用于开发测试。请使用以下命令运行：

``` bash
./profanity.x64 --matching profanity.txt -I 100 -s 1 -w 1
```

> 几点说明：

- 软件依赖于 `c++11`，所以机器需要安装 `xcode`。
- `mac` 机器跑这个更多是用于开发测试，运行时务必加上 `-w 1` 参数，否则生成私钥和地址可能存在不匹配的情况。
- `-I 100` 参数，可加快启动速度。
- `-s i` 用于跳过指定索引的设备，本人测试下来，部分型号 `mac` 集成的 `AMD` 类型显卡，生成的地址可能存在一些问题，因此需要跳过设备。

## 速度

本软件使用阿里云配置：`GPU 计算型 8 vCPU 32 GiB x 1 * NVIDIA V100`。速度在 `2.2亿 H/s` 左右：

<img width="100%" src="https://github.com/GG4mida/profanity-tron/blob/main/screenshot/demo.png?raw=true"/>

## 验证

生成的私钥和地址务必进行匹配验证。验证地址：[https://secretscan.org/PrivateKeyTron](https://secretscan.org/PrivateKeyTron)

## 安全

- 本软件基于 [profanity](https://github.com/johguse/profanity) 修改而来，原版程序存在私钥可爆破的漏洞，可参考：[Exploiting the Profanity Flaw](https://medium.com/amber-group/exploiting-the-profanity-flaw-e986576de7ab)

- 本软件已修复原版程序漏洞，代码如下：

```c++
cl_ulong4 Dispatcher::Device::createSeed()
{
#ifdef PROFANITY_DEBUG
	cl_ulong4 r;
	r.s[0] = 1;
	r.s[1] = 1;
	r.s[2] = 1;
	r.s[3] = 1;
	return r;
#else
	// Randomize private keys
  // Fix profanity seed create bug, ref: https://medium.com/amber-group/exploiting-the-profanity-flaw-e986576de7ab
	std::random_device rd;
	std::mt19937_64 eng1(rd());
	std::mt19937_64 eng2(rd());
	std::mt19937_64 eng3(rd());
	std::mt19937_64 eng4(rd());
	std::uniform_int_distribution<cl_ulong> distr;

	cl_ulong4 r;
	r.s[0] = distr(eng1);
	r.s[1] = distr(eng2);
	r.s[2] = distr(eng3);
	r.s[3] = distr(eng4);
	return r;
#endif
}
```

- 即使作者已声明软件无漏洞，但为了免去你的顾虑，请务必对生成的地址进行多签。参考：[https://sunyuchentron.medium.com/tronscan多重签名功能使用教程-一-3fb0cc8f6a0c](https://sunyuchentron.medium.com/tronscan多重签名功能使用教程-一-3fb0cc8f6a0c)

## 联系

- 电报: [@jackslowfak](https://t.me/jackslowfak)