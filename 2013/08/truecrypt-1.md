# TrueCrypt 使用经验[1]：关于加密算法和加密盘的类型 

-----

<div class="post-body entry-content">
　　自从两年前写了《<a href="../../2011/05/recommend-truecrypt.md">TrueCrypt——文件加密的法宝</a>》，陆续收到一些网友的来信或博客留言，针对 TrueCrypt 的使用提出了一些疑问。上周发了一篇博文《<a href="../../2013/07/online-backup-virtual-encrypted-disk.md">文件备份技巧：组合“虚拟加密盘”与“网盘”</a>》。又收到好几个读者的留言。所以今天再发一篇博文，汇总一下俺使用的经验，顺便也解答某些的常见提问。<br/>
　　如果你之前没用过 TrueCrypt，强烈建议你先看完《<a href="../../2011/05/file-encryption-overview.md">文件加密的扫盲介绍</a>》和《<a href="../../2011/05/recommend-truecrypt.md">TrueCrypt——文件加密的法宝</a>》这两篇。<br/>
<a name="more"></a><br/>
<br/>
<h2>★如何选择【加密算法】？</h2>
<br/>
　　先大致介绍一下 TrueCrypt 支持的加密算法和散列算法。然后再聊不同场合该如何选择。<br/>
<br/>
<h3>◇TrueCrypt 支持哪些加密算法？</h3>
<br/>
　　TrueCrypt 支持的加密算法有三种：AES、Twofish、Serpent。<br/>
　　AES 是洋文“Advanced Encryption Standard”的缩写。该算法原名是“Rijndael”，发布于2001年。2002年被美国官方确定为新的加密标准，用来替代之前的 DES（Data Encryption Standard）。当年美国政府采用公开征集的方式，参与海选的加密算法挺多；但进入最后一轮的算法只有5个，分别是：MARS、RC6、Rijndael、Serpent、Twofish（为避免纠纷，此处以字母序排列）<br/>
　　最终是 Rijndael 入选，所以如今经常提到的 AES 指的就是 Rijndael 算法。<br/>
<br/>
<h3>◇这三个算法的特点</h3>
<br/>
　　<b>规格</b><br/>
　　当年美国政府招标 AES 算法，有提出先决条件（限定加密算法的规格）：<br/>
1. 必须是“块加密”（又称“分组密码”，洋文叫“Block Cipher”）<br/>
2. 块长度必须是128位（128比特 = 16字节）<br/>
3. 密钥长度至少支持 128/192/256 位<br/>
4. 加密强度比 3DES（也叫“Triple-DES”） 高，并且速度比 3DES 快<br/>
　　因为存在这些竞标的约束，所以这三个算法都是块加密，也都支持 256位 的密钥长度（对同一个算法，密钥长度越大，加密强度越高）。<br/>
<br/>
　　<b>可靠性</b><br/>
　　在竞标 AES 的竞赛中，虽然 Twofish 和 Serpent 没有最终入选，但都是最后一轮的候选者。所以这三个算法至少在密码学层面，不会有明显的弱点。大伙儿可以放心地使用这三个算法。<br/>
<br/>
　　<b>强度</b><br/>
　　虽然这三个算法的规格差不多，但是在加密强度方面还是有差别的。下面是当年进入最后一轮竞标的5个候选算法，它们的“安全因子”（Safety Factor）对照表。限于篇幅，俺就不解释“安全因子”的含义了。你只需明白，安全因子越大，加密强度越高（越难破解）。<br/>
<br/>
<center><table border="1" cellpadding="5" cellspacing="0"><tbody>
<tr style="background:lightgrey;"><th>算法名称</th><th>安全因子</th><th>备注</th></tr>
<tr><td>MARS</td><td>1.90</td><td>&nbsp;</td></tr>
<tr><td>RC6</td><td>1.18</td><td>&nbsp;</td></tr>
<tr><td>Rijndael</td><td>1.56</td><td>它就是 AES 的最终获选者</td></tr>
<tr><td>Serpent</td><td>3.56</td><td>&nbsp;</td></tr>
<tr><td>Twofish</td><td>2.67</td><td>&nbsp;</td></tr>
</tbody></table></center>
<br/>
　　通过“安全因子”可知三者的加密强度是：Serpent ＞ Twofish ＞ Rijndael（AES）<br/>
<br/>
　　某些爱刨根问底的同学会纳闷了：既然 Rijndael 的安全因子不是最好，那为啥会中标捏？为啥不选强度最高的 Serpent？<br/>
　　因为“美国国家标准局”（NIST）在评估候选密码的时候，不是光看加密强度，还要考虑“性能、算法的复杂性，算法的扩展性”等诸多因素。Rijndael 虽然强度不是最高，但算法的灵活性强，而且比其它几个算法简单（越简单的算法，潜在的弱点就容易被发现，也更难隐藏“算法后门”）。<br/>
<br/>
<h3>◇加密算法的选择</h3>
<br/>
　　TrueCrypt 不光支持上述三个算法的“单重加密”，还支持多个算法组合成“双重加密”甚至“三重加密”。总共支持如下几种排列组合：<br/>
<blockquote>
AES<br/>
Serpent<br/>
Twofish <br/>
AES-Twofish<br/>
Serpent-AES<br/>
Twofish-Serpent<br/>
AES-Twofish-Serpent<br/>
Serpent-Twofish-AES</blockquote>
<br/>
　　显然，“三重加密”的强度最高，但是性能也最差。反之亦然。面对这么多选择，你需要考虑的就是：<b>如何平衡“安全性”＆“性能”。</b><br/>
　　“安全性”方面，刚刚已经介绍过了。至于“性能”，TrueCrypt 提供了一个 Benchmark 界面（在 TrueCrypt 主菜单点“Tools”然后再点“Benchmark”菜单项）。你可以在这个界面上测试不同组合的加密算法，其加密/解密的速度。请注意：在不同电脑测试，得到的排序结果是不同的。比方说有的 CPU 内置了 AES 指令，那么 AES 的速度就非常有优势（提升4至8倍）。想知道自己的电脑 CPU 是否支持 AES 指令，先打开 TrueCrypt 主界面，在主菜单点“Setting”，再点“Performance”就能看到。<br/>
<br/>
　　下面举几个例子，说说如何平衡“安全性”和“性能”。（举例中提到的“加密盘”，泛指“物理加密盘”和“虚拟加密盘”）<br/>
<br/>
　　举例1<br/>
　　如果你要制作的加密盘是用来备份个人的隐私或敏感文件，并且这个加密盘平时【很少用】，仅作为“灾难恢复”。那你完全可以选择强度最高的“三重加密”。<br/>
<br/>
　　举例2<br/>
　　如果你的加密盘是用来存放一些日常使用的文档类资料（比如电子书）。在这种情况下，磁盘【读】操作的频度远远高于【写】操作的频度。那么你就需要挑选“解密速度”快的组合。<br/>
　　（注：磁盘【读】操作对应“解密”，磁盘【写】操作对应“加密”）<br/>
<br/>
　　举例3<br/>
　　如果你使用 TrueCrypt 来加密系统分区，并且你的虚拟内存文件也在该分区。那么就要兼顾“读写两方面”的性能。这时候俺建议采用 AES 单重加密。如果你的 CPU 内置 AES 指令，更加要选 AES 相关的组合。<br/>
　　有的同学会担心，单重加密是不是不够强？俺的观点是：尽量【别】把敏感文件放到系统分区。这样的话，系统分区【没】啥要害，单重加密通常是够用滴。<br/>
<br/>
<br/>
<h2>★如何选择【散列算法】？</h2>
<br/>
　　“散列算法”又称为“哈希算法”（洋文叫 Hash）。不了解的同学，可以先看俺写的《<a href="../../2013/02/file-integrity-check.md">扫盲文件完整性校验——关于散列值和数字签名</a>》。这篇博文中大致解释了散列算法的概念。<br/>
　　用 TrueCrypt 创建加密盘，除了要选择加密算法的组合，还需要选择散列算法。TrueCrypt 需要使用散列算法来进行“伪随机数生成”，然后再利用“伪随机数”来产生加密卷的主密钥。<br/>
<br/>
　　TrueCrypt 支持如下三种散列算法：<br/>
<br/>
<center><table border="1" cellpadding="5" cellspacing="0"><tbody>
<tr style="background:lightgrey;"><th>算法名称</th><th>散列值比特数</th><th>散列值字节数</th></tr>
<tr><td>RIPEMD-160</td><td>160</td><td>20</td></tr>
<tr><td>SHA-512</td><td>512</td><td>64</td></tr>
<tr><td>Whirlpool</td><td>512</td><td>64</td></tr>
</tbody></table></center>
<br/>
　　通常而言，散列值的长度越大，发生“人为碰撞”和“随机碰撞”的概率就越小。所以俺一般选后两种。另外提醒一下：如果使用 TrueCrypt 加密【系统分区】，只能选 RIPEMD-160 这个散列算法。<br/>
<br/>
<br/>
<h2>★磁盘加密的【操作模式】</h2>
<br/>
　　（本小节涉及较多密码学的知识，不太懂技术的网友请自行跳过，以免浪费时间和心情）<br/>
<br/>
　　上周那篇《<a href="../../2013/07/online-backup-virtual-encrypted-disk.md">文件备份技巧：组合“虚拟加密盘”与“网盘”</a>》提到了：利用某些网盘客户端提供的“字节级差异同步”来优化加密盘的上传速度。<br/>
　　之后某网友在博客留言中提出质疑。他/她认为：加密盘中即使只修改了某个小文件，因为加密算法的特点，会导致加密卷文件中有大量的数据被修改。<br/>
　　这位网友思考问题比较深入，但对“块加密算法”只知其一，不知其二。下面俺稍微解释一下 TrueCrypt 采用的 XTS 操作模式。<br/>
<br/>
　　“块加密算法”因为每次加密的明文是固定大小的（也就是“块尺寸”），所以在实际使用场合中，需要结合具体的操作模式。所谓的操作模式，就是把明文通过某种方式分割成若干块，然后采用某种机制分别加密每一块。<br/>
　　最常听说的操作模式有 ECB、CBC、PCBC、CFB、等等（维基百科的“<a href="https://zh.wikipedia.org/wiki/%E5%9D%97%E5%AF%86%E7%A0%81%E7%9A%84%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F" rel="nofollow" target="_blank">这个页面</a>”有这些操作模式的原理介绍）。<br/>
<br/>
　　ECB 最简单，就是用密钥分别对 N 块明文进行加密，得到 N 块密文。此模式的最大缺陷在于：如果两块明文的内容完全一样，得到的那两块密文也会完全一样。所以 ECB 模式会遭遇“差分攻击”（通过【差异化分析】尝试进行破解）。<br/>
<br/>
　　后来 IBM 公司在70年代发明了 CBC——这个模式可以避免 ECB 的缺点。CBC 的原理也比较简单：先加密第一块明文，得到第一块的密文；然后在加密第二块明文的时候，把第一块密文作为输入参与运算；然后把第二块的密文作为第三块加密的输入；以此类推......<br/>
　　CBC 的好处在于，即使相邻两块明文的内容完全一样，加密得到的密文差异也极大。所以 CBC 可以对抗“差分攻击”。但是 CBC【没法】用于磁盘加密。因为 CBC 模式在加密/解密某个数据块的时候，要【连环依赖】于其它数据块的加密/解密。如果磁盘加密工具使用 CBC 模式，性能会非常非常差（原因很显然，自己想）。<br/>
<br/>
　　为了解决磁盘加密的需求，前几年又发明了新的模式——XEX 和 XTS（这两者差不多，XTS 是 XEX 的改良版）。使用 XTS 进行加密/解密，不同的数据块是完全独立的（解决了磁盘读写的性能问题）；另外，这玩意儿本身【没啥】明显弱点。所以，当今大部分磁盘加密工具（包括 TrueCrypt）都采用 XTS 模式。对该模式感兴趣的同学，可以看 TrueCrypt 手册的“Description of XTS mode”章节。俺这里就不细聊啦。<br/>
<br/>
<br/>
<h2>★【物理】加密盘 VS 【虚拟】加密盘</h2>
<br/>
　　“虚拟加密盘”也叫“virtual volume”，就是用一个大文件作为加密盘的容器。<br/>
　　俺在《<a href="../../2013/07/online-backup-virtual-encrypted-disk.md">文件备份技巧：组合“虚拟加密盘”与“网盘”</a>》中已经介绍过这两者常见的优缺点对比。今天除了说一下常见的优缺点对比，再稍微说一下【不太常见】的优缺点对比。<br/>
　　（物理加密盘其实又分为两种——加密普通分区 ＆ 加密系统分区。“加密系统分区”比较特殊，俺会在本系列的后续博文中专门介绍）<br/>
<br/>
<h3>◇“虚拟加密盘”的优点（同时也是“物理加密盘”的缺点）</h3>
<br/>
　　<b>优点1</b><br/>
　　首要的优点就是便于备份。因为“虚拟加密盘”说白了就是一个大文件，你只要把这个大文件 copy 到其它地方，就相当于完成了虚拟盘的备份。<br/>
　　而“物理加密盘”要备份，就相对麻烦一些。你需要使用专门的磁盘备份工具（比如 Ghost）。<br/>
<br/>
　　<b>优点2</b><br/>
　　虚拟加密盘可以嵌套任意多层（在一个虚拟盘内部再创建另一个虚拟盘）。<br/>
　　而物理加密盘不支持多重嵌套。（“外层卷”和“隐藏卷”不算嵌套。本系列的后续博文会单独介绍“隐藏卷”的经验）<br/>
<br/>
　　嵌套多层的好处在于，大大增加攻击者的破解难度。而且在一开始，攻击者根本无法知道你设置了多少层嵌套。（这个优点不明显。大部分网友应该用不到。但少数对安全性要求很高的网友，可能需要此优点）<br/>
<br/>
<h3>◇“物理加密盘”的优点（同时也是“虚拟加密盘”的缺点）</h3>
<br/>
　　<b>优点1</b><br/>
　　因为你没法把整个操作系统的系统分区放到一个虚拟加密盘中。所以，对系统分区只能采用“物理加密盘”的方式，无法采用“虚拟加密盘”的方式。<br/>
　　支持加密【系统分区】，这是“物理加密盘”主要的优点之一。对【系统分区】的加密很重要，在<a href="../../2013/08/truecrypt-3.md">本系列第3篇</a>会讨论与之相关的话题。<br/>
<br/>
　　<b>优点2</b><br/>
　　其实“物理加密盘”还有一个额外的好处，但大多数人碰不到这种情形。<br/>
　　当你干了某些事情，并被警方盯上。警察可能会收缴你的电脑并拿回去做“取证分析”（如今公安部的省市级机构，都有“信息取证”的技术人员）。假如你用的是“虚拟加密盘”，那么你的电脑中就会有一些大文件（加密盘的卷文件）。当警察盘问你，这些个大文件是干嘛用滴？你很难解释，也很难抵赖。<br/>
　　反之，如果你用的是“物理加密盘”，当加密分区还【没】被挂载时，分区中的数据是杂乱无章的，就像是随机数据。那么你就可以辩解说，这个分区是【尚未格式化】滴。（注：这种辩解可以骗过外行，但不一定能骗过【内行】；至于如何对付【内行】，<a href="../../2013/10/truecrypt-4.md">本系列第4篇</a>会有专门的讨论）<br/>
<br/>
<h3>◇咋样结合两者的优点？</h3>
<br/>
　　这其实很容易想到的——就是把虚拟加密盘放到物理加密盘内部。当然，如果放到物理加密盘的【隐藏卷】部分，效果就更好啦。（因为篇幅的关系，“隐藏卷”的经验放到下一篇，本文先不讨论）<br/>
<br/>
　　把虚拟加密盘放到物理加密盘内部，你的操作系统中就不会有可疑的大文件，而且不影响虚拟加密盘的备份。真可谓一举多得，当然也有缺点——主要的缺点就是性能会下降。好在如今的电脑，CPU 已经很强劲了，动不动就是好几个核。所以，这点性能下降还是可以接受滴。<br/>
<br/>
<br/>
<h2>★【物理】加密盘的两种玩法——全盘加密 VS 逐个分区加密</h2>
<br/>
　　使用物理加密盘可以有两种搞法：其一，全盘加密；其二，逐个分区加密。<br/>
　　对这两种方案，俺个人建议采用后者——逐个分区加密。把整个硬盘做一次性的全盘加密，虽然操作起来稍微简单一些，但是存在如下一些隐患。<br/>
<br/>
<h3>◇隐患1——强度不够</h3>
<br/>
　　大多数电脑都是单硬盘，也就是说系统分区也在该硬盘上。所以，对单硬盘 PC 的“全盘加密”也就是“加密系统分区”。而 TrueCrypt 在加密系统分区的时候，仅支持口令认证，不支持 KeyFiles 认证。这就导致加密强度【大打折扣】——因为 KeyFiles 难以用暴力穷举，而口令更容易被暴力穷举。<br/>
<br/>
<h3>◇隐患2——灵活性不够</h3>
<br/>
　　如今的硬盘都很大，里面通常会有不止一个分区。不同分区存放的数据往往也不同。有的分区存放的数据重要，有的不重要。<br/>
　　如果你对整个硬盘进行一次性加密，就面临一个两难——如何选择加密算法？加密算法太强，性能会很差；加密算法不够强，安全性又不够。<br/>
　　而如果采用不同分区逐个加密的方案，就可以避免上述难点。你可以对特别敏感的分区采用更强的加密算法（但也更慢），对不太重要的分区采用弱一些（但也快一些）的加密算法。<br/>
<br/>
<h3>◇隐患3——兼容性不够</h3>
<br/>
　　假如你的硬盘安装了多个不同的操作系统（多引导）。这种情况下使用全盘加密可能会出现一些冲突，导致某些系统无法启动（Linux 与 Windows 混装的环境尤其明显）。<br/>
<br/>
<br/>
<h2>★未完待续</h2>
<br/>
　　本想一篇博文写完。写到这里发现内容已经很长了。今天先发出来，剩余部分再发2到3篇。大伙儿请见谅。<br/>
<br/>
<b><a href="../../2011/05/recommend-truecrypt.md#index">回到本系列目录</a></b><br/>
<br/>
<br/>
<b>俺博客上，和本文相关的帖子（需翻墙）</b>：<br/>
《<a href="../../2011/05/recommend-truecrypt.md">TrueCrypt——文件加密的法宝</a>》<br/>
《<a href="../../2015/10/VeraCrypt.md">扫盲 VeraCrypt——跨平台的 TrueCrypt 替代品</a>》<br/>
《<a href="../../2019/02/Use-Disk-Encryption-Anti-Computer-Forensics.md">如何用“磁盘加密”对抗警方的【取证软件】和【刑讯逼供】，兼谈数据删除技巧</a>》<br/>
《<a href="../../2011/05/file-encryption-overview.md">文件加密的扫盲介绍</a>》<br/>
《<a href="../../2013/02/file-integrity-check.md">扫盲文件完整性校验——关于散列值和数字签名</a>》
</div>


------------------------------------------------

版权声明本博客所有的原创文章，作者皆保留版权。转载必须包含本声明，保持本文完整，并以超链接形式注明作者编程随想和本文原始地址：https://program-think.blogspot.com/2013/08/truecrypt-1.html
