# 如何用 Tor 访问对 Tor 不友好的网站——扫盲“三重代理”及其它招数 

-----

<div class="post-body entry-content">
　　最近几个月，有不少读者反馈“使用 Tor 碰到的问题”。比如说：访问 Startpage 搜索引擎，经常要输入验证码（以前不需要）。还有一些读者抱怨说，用 Tor Browser 在博客留言，比以前更困难（“人机验证”要搞很久）<br/>
　　有鉴于此，今天专门发一篇教程，比较【系统性】地介绍若干种解决方法。<br/>
<a name="more"></a><br/>
<center><img alt="不见图 请翻墙" src="images/UEJSz_vaNCCnJZ0vFEbAM0P2etlEwSFOOC-dDI9_55uIKQElo0ewEUREdbLpGvULyr_ran6G0B8oVh2rJIy4NBaOnL9znEaDeNKh15geLSNe4CVCPlbxhSRY2oYz"/></center>
<br/>
<h2>★本文的目标读者</h2>
<br/>
　　本文主要针对【Tor 的重度用户】，教你如何在“全程使用 Tor”这个前提下，依然能够访问那些“对 Tor【不】友好的网站”。<br/>
　　既然目标读者是【Tor 的重度用户】，在阅读本文之前，俺假定你已经掌握了如下这些招数：<br/>
<blockquote style="background-color:#DDD;">
代理的基本概念<br/>
如何设置浏览器的 SOCKS 代理<br/>
Tor 的基本使用方式（可以是“Tor Browser”，也可以是“裸 Tor”）<br/>
Tor 的基本原理（三级跳、入口节点、出口节点 ......）<br/>
如何给 Tor 添加【前置代理】<br/>
虚拟机的基本使用方式</blockquote>
　　如果你对上述的某几项还【不】太熟悉，先看本博客的相关教程（如下）：<br/>
《<a href="../../2013/11/tor-faq.md">“如何翻墙”系列：关于 Tor 的常见问题解答</a>》<br/>
《<a href="../../2009/09/break-through-gfw-with-tor.md">戴“套”翻墻的方法</a>》<br/>
《<a href="../../2018/04/gfw-tor-browser-7.5-meek.md">“如何翻墙”系列：扫盲 Tor Browser 7.5——关于 meek 插件的配置、优化、原理</a>》<br/>
《<a href="../../2015/03/Tor-Arm.md">扫盲 Arm——Tor 的界面前端（替代已死亡的 Vidalia）</a>》<br/>
《<a href="../../2012/10/system-vm-0.md">扫盲操作系统虚拟机</a>》（系列）<br/>
<br/>
<br/>
<h2>★名词解释</h2>
<br/>
<h3>◇代理</h3>
<br/>
　　【广义的】代理，既包括大伙儿日常所说的代理（proxy），也包括 VPN。<br/>
　　VPN 可理解为【系统级】的代理软件——运行了 VPN 之后，操作系统中的每个上网软件都自动通过 VPN 的线路联网。<br/>
　　在本文中，俺所说的“代理”指的是【广义的】代理（含 VPN）。如果俺要特指【除了】VPN 之前的其它代理软件，俺会使用“传统代理”这个称呼。<br/>
<br/>
<h3>◇Tor</h3>
<br/>
　　在本文中，俺所说的“Tor”，既包括了“Tor Browser”，也包括“裸 Tor”。如果需要特指其中某种，俺会专门指出。<br/>
<br/>
<br/>
<h2>★原理分析——为啥有些网站/软件，对 Tor 不友好？</h2>
<br/>
　　要解决“对 Tor 不友好”这个问题，你首先需要了解【原理】——为啥有些网站对 Tor 不友好。<br/>
　　俺针对不同的情况，分别解释其原因：<br/>
<br/>
<h3>◇情况1：【网站】屏蔽了 Tor 的出口节点</h3>
<br/>
　　这种情况很好理解——有些网站讨厌“Tor 匿名网络”，一旦该网站的服务器判断出——访问者 IP 是“Tor 出口节点”，就【不】处理来自这个 IP 的网络协议。<br/>
<br/>
<h3>◇情况2：【GFW】屏蔽了 Tor 的出口节点</h3>
<br/>
　　很多网民只知道：GFW 会屏蔽“墙内对墙外的访问”。其实 GFW 的系统设计是【双向过滤】。也就是说，GFW 既可以屏蔽“墙内访问墙外”，也可以屏蔽“墙外访问墙内”。<br/>
　　由于 Tor 是 GFW 的眼中钉，GFW 当然也会把 Tor 网络中的【出口节点】列入其“IP 黑名单”。也就是说，你用 Tor 去访问【墙内】网站，【有可能】被 GFW 拦截。<br/>
　　为啥俺要强调【有可能】三个字？因为这里面存在几个变数——<br/>
1. 漏网的“出口节点”<br/>
因为 Tor 全球网络的“出口节点”会【动态】变化，GFW 的“IP 黑名单”可能会有漏网。<br/>
2. CDN 或镜像服务器<br/>
有些国内网站可能“启用了 CDN”或“在墙外有镜像服务器”，那么 Tor 依然可以正常访问这些墙内网站（实际上 Tor 访问的是这些网站的【墙外】镜像或缓存）。<br/>
<br/>
<h3>◇情况3：某个 Tor 出口节点对网站的访问流量过大，被网站【动态】拉黑</h3>
<br/>
　　大多数 Tor 的重度用户碰到的是这第三种情况——<br/>
很多大型网站的服务器都有一些“安全防护措施”。比如说，网站服务器会统计每个公网 IP 的访问量。一旦发现某个公网 IP（对网站）的访问量太大，就认定是某种【恶意行为】（网站服务器会怀疑——这么大的访问量来自同一个 IP，更像是某种“机器行为”，而不像是“人的行为”）<br/>
　　举例：<br/>
　　当你通过 Tor 代理去使用 Google 搜索，多半会在界面上看到诸如此类的提示，并要求你进行【人机验证】：<br/>
<blockquote style="background-color:#DDD;">
Our systems have detected unusual traffic from your computer network. This page checks to see if it's really you sending the requests, and not a robot.</blockquote>
　　为啥这种事情会发生在 Tor 身上捏？<br/>
　　其根本的原因在于——Tor 的全球网络虽然有成百上千的节点，但大部分节点都【不是】“出口节点”。或者说，“出口节点”只占 Tor 全球网络节点数的一个较小的比例。而 Tor 的全球用户数是逐年增长，“用户数的增长”明显快于“出口节点数的增长”。<br/>
　　其结果就是——每个出口节点的【对外访问量】越来越高。因此，当你用 Tor 去访问某些网站（比如刚才提到的“Google 搜索”），就会被当成是“恶意行为”，并要求你进行“人机验证”。<br/>
<br/>
<br/>
<h2>★需求分析——Tor 的使用场景</h2>
<br/>
　　接下来，咱们来谈谈：Tor 用户通常会拿 Tor 来干啥。<br/>
　　为啥要谈“需求分析”捏？因为不同的使用场景，解决的方法也不同。<br/>
<br/>
<h3>◇使用场景1：对 Web 页面的【只读式】访问</h3>
<br/>
　　所谓的“【只读式】访问”，顾名思义——你只是看这个网页的内容。<br/>
　　对这种方式，只要你能想办法（通过 Tor）把网页内容加载到你的浏览器中，就达到目的啦。<br/>
<br/>
<h3>◇使用场景2：对 Web 页面的【交互式】访问</h3>
<br/>
　　这里所说的“【交互式】”是相对于前面“场景1”所说的“【只读式】”。<br/>
　　为了对比这两种场景，拿俺博客来举例——<br/>
如果你只是（通过 Tor）看俺的博文，这属于“场景1——只读式”；<br/>
如果你想要（通过 Tor）在博客评论区留言，这属于“场景2——交互式”。<br/>
<br/>
<h3>◇使用场景3：使用某个【上网软件】</h3>
<br/>
　　刚才聊的“场景1 ＆ 场景2”都是针对【浏览器】的使用。<br/>
　　但有些时候，你可能需要通过“Tor 代理”的方式，使用其它的一些桌面软件（比如：聊天工具、下载工具、等等）。<br/>
<br/>
<br/>
<h2>★解决方法1：网页快照</h2>
<br/>
<h3>◇原理</h3>
<br/>
　　某些网站提供了“网页快照”的功能（比如：很多搜索引擎都提供了这个功能）。<br/>
　　假设 A 网站提供了“网页快照”的功能，并且对 Tor 友好；而 B 网站对 Tor【不】友好。那么，你可以（通过 Tor 代理）在 A 网站上访问 B 网站的网页快照。<br/>
<br/>
<h3>◇使用方法</h3>
<br/>
　　<b>网页快照的方法——搜索引擎</b><br/>
　　其使用步骤如下：<br/>
1. 找一个【对 Tor 友好】的搜索引擎，并且要支持“网页快照”的功能。<br/>
2. 直接在搜索框里面搜“你想要访问的【网址】”。点了搜索按钮之后，该网址对应的网页通常会出现在搜索结果的第一条。<br/>
3. 这时候，你点击该网页的快照，该搜索引擎就会把这个网页的内容显示给你看。<br/>
　　通过上述步骤，你访问的是“搜索引擎的服务器”，而【不是】目标网站的服务器。哪怕目标网站屏蔽了 Tor，对你也没影响。<br/>
<br/>
　　刚才说了，使用这招，搜索引擎必须“对 Tor 友好”并且“支持网页快照”。符合这两条的，【至少】有 <a href="https://www.startpage.com/" rel="nofollow" target="_blank">Startpage</a> 和 <a href="https://www.bing.com/" rel="nofollow" target="_blank">Bing</a>。<br/>
　　Startpage 这个网站，俺专门介绍过（参见“<a href="../../2018/11/Private-Search-Engine-Startpage.md">这篇教程</a>”）。它对“隐私保护”做得比较好，而且其后台是通过 Google 获得搜索结果。因此，它的搜索质量与 Google 基本等同。<br/>
　　原先，Startpage一直对 Tor 很友好；但全球疫情之后，也开始变得不太友好（出现人机验证的次数变多）。俺个人的猜测是：全球疫情使得很多国家的网民更多地宅在家中，全球网民的平均上网时间变多了；相应的，全球的 Tor 流量也变大了，Tor 网络对 Startpage 访问量也变大了。当 Tor 网络的流量大到一定程度，可能超出了 Startpage 服务器上设定的某个临界值。于是 Startpage 也开始对 Tor 用户显示“人机验证”了。<br/>
<br/>
　　<b>网页快照的方法——互联网档案馆</b><br/>
　　除了“搜索引擎”这招，另一个“网页快照”的方法是【互联网档案馆】。该网站长期对 Tor 友好，而且它能够对【同一个网址】保存 N 份快照。其使用步骤如下：<br/>
1. 打开“<a href="https://web.archive.org/" rel="nofollow" target="_blank">这个页面</a>”，上方有一个搜索框；<br/>
2. 输入你想要查看的【网址】，敲回车；<br/>
3. 它会显示一个日历，日历上做了标记的日期，就说明——在那个时间点保存有该网址的快照。<br/>
4. 把鼠标移到【做了标记的日期】上面，你就可以看到——你想要访问的网址，在“某年某月某日 某时某分某秒”所保存的快照。<br/>
<br/>
　　“互联网档案馆”上的网页快照，很多都是用户手动生成滴。如果你想要看的网址比较【冷门】，“互联网档案馆”还【没有】该网址的快照，你可以自己生成一个快照，步骤如下：<br/>
1. 打开“<a href="https://web.archive.org/" rel="nofollow" target="_blank">这个页面</a>”；<br/>
2. 找到 <q style="background-color:#DDD;">Save Page Now</q> 字样，下面有一个编辑框，输入你想要看的网址；<br/>
3. 点击“SAVE PAGE”按钮<br/>
4. 稍等片刻，“互联网档案馆”就会把这个网址抓到它服务器上永久保存，同时会把该网页的快照显示给你看。<br/>
<br/>
<br/>
<h3>◇优点</h3>
<br/>
　　<b>优点1</b><br/>
　　【无需】安装任何软件，也无需改动 Tor 的任何配置或者系统设置。<br/>
<br/>
　　<b>优点2</b><br/>
　　前面章节提到“对 Tor 不友好的三种情况”，这招都能搞定。<br/>
<br/>
<h3>◇缺点</h3>
<br/>
　　前面谈需求分析，提到“三种使用场景”，这招只适用于“【只读式】Web 访问”。<br/>
<br/>
<br/>
<h2>★解决方法2：Web 代理</h2>
<br/>
<h3>◇原理</h3>
<br/>
　　网页快照的方式，通常会有些【滞后】（不够新鲜）。如果你想要看到【实时的】网页内容，就得考虑“Web 代理”。<br/>
　　所谓的“Web 代理”，有时候又称作“网页代理”。顾名思义，其“代理功能”是在 Web 界面内完成，不需要安装额外的代理软件。<br/>
　　假设你找到一个对 Tor 友好的“Web 代理”，就可以用它来【实时查看】那些对 Tor【不】友好的网站的页面。<br/>
<br/>
<h3>◇使用方法</h3>
<br/>
　　随便找个搜索引擎，搜一下 <code>web proxy</code> 就可以找到一大堆各式各样的“网页代理”。<br/>
　　挑选一个对 Tor 友好的，就可以用啦。<br/>
<br/>
<h3>◇优点</h3>
<br/>
　　<b>优点1</b><br/>
　　【无需】安装任何软件，也无需改动 Tor 的任何配置或者系统设置。<br/>
<br/>
　　<b>优点2</b><br/>
　　前面章节提到“对 Tor 不友好的三种情况”，这招都能搞定。<br/>
<br/>
　　<b>优点3</b><br/>
　　如果拿这招与“网页快照”对比——这招相当于“正在进行时”，而“网页快照”属于“过去时”。<br/>
<br/>
<h3>◇缺点</h3>
<br/>
　　前面谈需求分析，提到“三种使用场景”，这招只适用于“【只读式】Web 访问”。<br/>
<br/>
<br/>
<h2>★解决方法3：手动切换 Tor 线路——动态切换【出口节点】</h2>
<br/>
<h3>◇原理</h3>
<br/>
　　前面的章节介绍过“对 Tor 不友好的三种情况”。这招专门针对第3种情况——“出口节点”被目标网站动态拉黑。<br/>
　　当你用 Tor 访问某个网站，网站提示你进行【人机验证】。这通常说明——你正在使用的出口节点已经被目标网站列入“流量过大的黑名单”。为了解决这个问题，你可以“手动切换 Tor 线路”。当线路切换了，你正在使用的【出口节点】也就变了。<br/>
<br/>
<h3>◇配置方法</h3>
<br/>
　　不论是“Tor Browser”还是“裸 Tor”都提供了“手动切换 Tor 线路”的功能。下面分别说：<br/>
<br/>
　　<b>裸 Tor</b><br/>
　　对使用“裸 Tor”的同学，可以通过 Arm/Nyx 这两款“Tor 管理界面”来切换 Tor 的三级跳线路。<br/>
　　关于 Arm 的教程，俺在多年前（2015）已经写过一篇《<a href="../../2015/03/Tor-Arm.md">扫盲 Arm——Tor 的界面前端（替代已死亡的 Vidalia）</a>》。至于 Nyx，是 Arm 的衍生品——Arm 依赖 Python 2.x（不支持 Python 3.x），而 Nyx 支持 Python 3.x 版本。Arm 与 Nyx 两者的使用基本类似（都可以看上述这篇教程）。<br/>
　　以下这张截图摘自上述那篇 Arm 使用教程，截图中的下拉菜单中有 <q style="background-color:#DDD;">New Identity</q> 这项，可以让你手动切换“三级跳的线路”。<br/>
<br/>
<center><img alt="不见图 请翻墙" src="images/_I4xchBopbUJ5-QVlkBxBcbOkY01FIy4g1W5IWRIIQSleFOW_VKchV1V9NfI7bnlyofy3e_c3V8m3gEkAxna6CtOQ5kH3mSJ-3dUW0D_8nMQwG-xciBeTs3ScdvJOGwCrtMg-U5RhQ"/></center>
<br/>
　　<b>Tor Browser</b><br/>
　　对于使用 Tor Browser 的同学，可以使用其界面上的某个下拉菜单（截图如下）<br/>
<br/>
<center><img alt="不见图 请翻墙" src="images/S9xUPrUTk8FUw0s9l18JQvxftEYlHAYFzP_YAo3CZz3k8GcG63NEmZ7k8HNDIOuH9PRt258MT1A03qdWWvdYmw-SmkfVcaPVq4TZ3n4Ih2pbqFyM2RNxGUadysN7Pn8k-Ic_8t55NJU"/></center>
<br/>
　　请注意菜单中的这两项：<q style="background-color:#DDD;">New Identity</q> ＆ <q style="background-color:#DDD;">New Tor Circuit for this Site</q>。这2项都可以用来切换 Tor 线路，两者有啥差别捏？请看 Tor 官网的帮助页面（链接在“<a href="https://support.torproject.org/tbb/tbb-29/" rel="nofollow" target="_blank">这里</a>”）。<br/>
<br/>
<h3>◇优点</h3>
<br/>
　　<b>优点1</b><br/>
　　【无需】安装任何软件，也无需改动 Tor 的任何配置或者系统设置。<br/>
<br/>
　　<b>优点2</b><br/>
　　前面谈需求分析，提到“三种使用场景”，这招都能搞定。<br/>
<br/>
<h3>◇缺点</h3>
<br/>
　　<b>缺点1</b><br/>
　　在“对 Tor 不友好的三种情况”中，这招只能解决“第3种情况”——某个出口节点被目标网站【动态拉黑】。<br/>
　　另外两种情况，用这招是【没戏】滴！<br/>
<br/>
　　<b>缺点2</b><br/>
　　Tor 出口节点是【随机选取】滴。也就是说，用了这招是否有效，全凭运气。<br/>
　　如果你碰巧切换到某个【流量较少】的出口节点，用这个出口节点去访问目标网站，目标网站就【不会】把该节点视作“恶意行为”。<br/>
　　请注意：在同一个瞬间，同一个出口节点对不同网站的访问量，也是不同滴。假设某个出口节点对 A 网站的流量很大，对 B 网站的流量很小。而你要访问的是 B 网站，那么你切换到这个出口节点，就【不会】被 B 网站拉黑。<br/>
<br/>
<br/>
<h2>★解决方法4：限制 Tor 出口节点所属的【国别】</h2>
<br/>
<h3>◇原理</h3>
<br/>
　　这招有点类似于“解决方法3”。“解决方法3”算是【随机碰运气】，而这招则提供了某种更有针对性的方法。<br/>
　　一般来是，Tor 网络的出口节点，在不同国家中的分布是【不平均】滴，流量也是不平均滴。具体到某个网站，有可能“A 国的出口节点”对该网站的流量比较大，而“B 国的出口节点”对该网站的流量比较小。<br/>
　　基于此，你可以修改 Tor 的配置文件（<code>torrc</code>），限定 Tor 只使用 B 国的出口节点，就可以尽量降低“被目标网站动态拉黑”的概率。<br/>
<br/>
<h3>◇配置方法</h3>
<br/>
　　如何通过修改 <code>torrc</code> 来限制“出口节点所在的国别”捏？请看教程《<a href="../../2013/11/tor-faq.md">“如何翻墙”系列：关于 Tor 的常见问题解答</a>》<br/>
<br/>
　　看到这里，某些同学肯定会问了——哪个国家的出口节点，流量会比较小？<br/>
　　很遗憾，这个问题【没有】标准答案——<br/>
其一，不同的网站，Tor 流量的“国别分布”，可能不同。<br/>
其二，同一个网站，随着时间推移，其 Tor 流量的“国别分布”也可能发生变化。<br/>
　　所以，如果你要用这招，只能靠自己慢慢摸索。你可以每次限定一个国家，然后用上几天。逐步观察之后，自然就能感觉出——用哪个国家的“出口节点”访问你常去的网站，碰到“人机验证”的概率比较小。<br/>
<br/>
<h3>◇优点</h3>
<br/>
　　<b>优点1</b><br/>
　　只需要修改 Tor 的配置，【无需】安装任何软件，也无需修改其它系统设置。<br/>
<br/>
　　<b>优点2</b><br/>
　　前面谈需求分析，提到“三种使用场景”，这招都能搞定。<br/>
<br/>
<h3>◇缺点</h3>
<br/>
　　<b>缺点1</b><br/>
　　在“对 Tor 不友好的三种情况”中，这招只能解决“第3种情况”——某个出口节点被目标网站【动态拉黑】。<br/>
　　另外两种情况，用这招是【没戏】滴！<br/>
<br/>
　　<b>缺点2</b><br/>
　　你需要花时间去摸索，找到不同国别之间的流量差异。而且不同的目标网站，国别之间的流量差异，也会不同。<br/>
　　也就是说，用这招比较劳神。<br/>
<br/>
<br/>
<h2>★解决方法5：三重代理——给 Tor 添加【后置代理】</h2>
<br/>
　　前面所说的几种解决方法，都有各自的缺陷。稍微总结一下：<br/>
　　<b>方法1 ＆ 方法2</b><br/>
　　在【网络】这个维度是 OK 滴（三种对 Tor 不友好的情况，都能搞定）；但在【使用场景】这个维度有缺陷——只能用于“网页的【只读式】访问”，对于网页的【交互式】访问，或者对于【桌面上网软件】，这两招就【不灵】啦。<br/>
<br/>
　　<b>方法3 ＆ 方法4</b><br/>
　　在【使用场景】这个维度是 OK 滴（三种使用场景，都可以搞定）；但在【网络】这个维度有缺陷——只能针对那些“动态拉黑”的情况（参见“原理分析”的情况3），对那些“显式屏蔽 Tor 的网站”就【不灵】啦（参见“原理分析”的情况1＆情况2）。<br/>
<br/>
　　虽然前面4个招数都有各自的缺陷，不过捏，大伙儿别担心！<br/>
　　为了解决上述4种方法的弊端，俺隆重推出终极大杀器——三重代理。他可以同时搞定“三种使用场景”，也可以同时搞定“对 Tor 不友好的三种情况”。<br/>
<br/>
<h3>◇（老调重弹）Tor 的双重代理，及其重要性</h3>
<br/>
　　在正式聊“三重代理”之前，请允许俺稍微唠叨一下。以下这几段话，在近几年的很多篇旧博文中已经提到过多次。今天再次重复，让列位看官加深印象。<br/>
　　一般来说，使用 Tor 的用户都会拿“其它翻墙工具”作为 Tor 的【前置代理】。这主要是因为 GFW 屏蔽了 Tor 的全球网络（GFW 把所有的 Tor 节点/网桥都加入“IP 黑名单”）。通过给 Tor 增加【前置代理】，就可以让 Tor 突破 GFW 的封锁。<br/>
　　后来（大约2014年），Tor 社区研发了【meek 插件】，使得 Tor 可以直接突破 GFW 的封锁。有了 meek 插件之后，其实 Tor 已经可以摆脱“前置代理”的依赖，独立联网。但俺依然推荐 Tor 用户使用【前置代理】。更进一步地说，即使你人在【墙外】（无需翻墙），俺依然建议你在使用 Tor 的使用，加一个【前置代理】。<br/>
　　其好处【至少】包括如下：<br/>
　　<b>好处1</b><br/>
　　当你给 Tor 配置了（加密的）前置代理，一旦你的运营商监控你的网络流量，看到的是【前置代理】的加密流量，也就【无法】知道你在使用 Tor。<br/>
　　对于那些非常看重安全性（隐匿性）的同学，你既要使用匿名网络（Tor 或 I2P），又【不能】让别人（政府/警方）知道你在使用匿名网络。<br/>
　　<b>好处2</b><br/>
　　任何软件都可能出现安全漏洞，Tor 的协议栈也不例外。万一出现这种情况，并且你【没】给 Tor 设置“前置代理”，那么，能够监控你流量的攻击者就有可能利用该漏洞，并破解你的 Tor 加密。而一旦破解了 Tor 的加密，攻击者就会看到你真实的上网流量。<br/>
　　反之，如果你用双重代理，不论是“前置代理出现安全漏洞”还是“Tor 出现安全漏洞”，攻击者都【无法】看到你的真实上网流量。而“Tor”与“前置代理”【同时出现】安全漏洞的概率非常低，基本上可以忽略不计。<br/>
<br/>
<h3>◇原理</h3>
<br/>
　　所谓的“三重代理”，也就是给 Tor 增加一个【后置代理】。<br/>
　　为了便于理解，俺特地画了“双重代理”与“三重代理”的数据流示意图。通过两种方式的对比，可以更好地帮你理解“三重代理”是如何运作滴。<br/>
<br/>
<center><img alt="不见图 请翻墙" src="images/dHNXCMA2uZddawgGk3kUkQFmq6sFe3ETElYWOLyvVp4p7zf6FDvq2nZnuRgZ5wrey9ajv87GlbaOW1QWVFACsw8S-sE9VxSpDAu22WVq_CjrkOJf5ffopJsrKxYXYpJSys0BBAAAeCE"/><br/>
（【双重代理】数据流示意图）</center>
<br/>
<center><img alt="不见图 请翻墙" src="images/z8FD-Olj5MZeYZ6GleVmrUR8mORcekvP6RGjD7a6uWcarw29sbCfzUR-Tiujy-l5rI27_UR35jLnp7mlbe1yHoX2_PjkFnInIlGivnslrL0D7uM0ixqDv6Yxdbcwp889jurI5BhDMuI"/><br/>
（【三重代理】数据流示意图）</center>
<br/>
　　从上述示意图可见——<br/>
当你使用“双重代理”时，目标网站看到的“访问者 IP”是【Tor 出口节点】的 IP；<br/>
当你使用“三重代理”时，目标网站看到的“访问者 IP”是【后置代理的服务器】的 IP。<br/>
　　也就是说，采用了“三重代理”之后，目标网站【无法】看出你在使用 Tor。开篇提到“对 Tor 不友好的三种情况”，用“三重代理”这招都能搞定。同时，在“需求分析”中提到的“三种使用场景”，“三重代理”这招也都能搞定。<br/>
<br/>
<h3>◇配置方法</h3>
<br/>
　　<b>第1步——先搞定“双重代理”</b><br/>
　　要玩转“基于 Tor 的三重代理”之前，你首先要能玩转“基于 Tor 的双重代理”。具体该如何搞，在如下的系列教程中已经写得很详细了（参见这个系列的第5篇）<br/>
《<a href="../../2010/04/howto-cover-your-tracks-0.md">如何隐藏你的踪迹，避免跨省追捕</a>》<br/>
<br/>
　　<b>第2步——选择“后置代理”的软件</b><br/>
　　在这一步，你要选择一款翻墙工具作为“后置代理”。<br/>
　　【不是】每一种翻墙工具都能作为“后置代理”。这里有一个前提条件——<b>作为“后置代理”的翻墙工具，它本身要提供某个配置界面，使得你能够指定它走别的代理进行联网。</b><br/>
　　考虑到有些读者是技术菜鸟，俺再说得更浅显一些。<br/>
　　所有的翻墙代理，都可以帮你把浏览器的流量转发到它自己的服务器，从而实现翻墙。在这种情况下，翻墙软件【直连】自己的服务器。俺称之为【直连模式】（所有的翻墙软件都支持这种模式）。<br/>
　　除了上述所说的【直连模式】，还有一种【代理模式】（有的翻墙软件支持，有的不支持）。所谓的【代理模式】是指——翻墙软件【不】直接连接自己的服务器，而是通过另一个代理去连接自己的服务器。为了使用别的代理，那么翻墙软件的界面上就必须提供某个“设置代理”的界面。<br/>
　　<b>如果你想要拿某个翻墙软件作为 Tor 的“后置代理”，该软件必须支持【代理模式】。</b><br/>
<br/>
　　<b>第3步——让“后置代理”通过 Tor 联网</b><br/>
　　在这一步，需要让“后置代理”通过 Tor 联网。<b>在整个“三重代理”的配置流程中，这一步最容易搞砸。</b><br/>
　　由于 Tor 默认提供的是 SOCKS 代理，而“后置代理”【不一定】支持走 SOCKS 联网，下面分两种情况讨论（3.1 ＆ 3.2）<br/>
<br/>
　　<b>3.1 “后置代理”【支持】走 SOCKS 代理联网</b><br/>
　　在“后置代理”的软件界面上，找到“代理设置”（或诸如此类）的配置界面，在上面填写 Tor 的 IP 与端口。<br/>
　　如果 Tor 与后置代理在同一个系统，IP 就填写 <code>127.0.0.1</code>；如果 Tor 在另一个操作系统，就填写另一个操作系统的 IP。<br/>
　　对于“裸 Tor”，端口号填写 <code>9050</code>；对于“Tor Browser”，端口号填写 <code>9150</code><br/>
　　如果是【跨主机使用 Tor】，你还需修改 Tor 的配置文件（<code>torrc</code>），使得 Tor 的 SOCKS 监听端口绑定到地址 <code>0.0.0.0</code> 上。具体如何修改，参见《<a href="../../2013/11/tor-faq.md">关于 Tor 的常见问题解答</a>》。<br/>
<br/>
　　<b>3.2 “后置代理”【不支持】走 SOCKS 代理联网，只支持 HTTP 代理联网</b><br/>
　　对这种情况，你还需要再折腾一下“HTTP 代理”转“SOCKS 代理”。俺推荐用 Privoxy 工具来干这事儿。<br/>
　　使用 Privoxy 中转之后，“后置代理”的“代理配置界面”就【不能】填 Tor 的 IP 及端口，而要填 Privoxy 的 IP 及端口。然后你还要再修改 Privoxy 的配置文件，使得 Privoxy 把收到的 HTTP 请求转发到 Tor 的 SOCKS 端口。（具体如何搞，参见教程《<a href="../../2014/12/gfw-privoxy.md">如何用 Privoxy 辅助翻墙？</a>》）。<br/>
<br/>
　　<b>3.3 如何验证这一步的配置正确？</b><br/>
　　通过下面两个方式来验证：你已经配置正确。<br/>
　　首先，让 Tor 启动并联网，然后再启动“后置代理”，“后置代理”也必须能正常联网。<br/>
　　其次，把 Tor 关闭，然后再重启“后置代理”，“后置代理”【无法】联网。<br/>
　　如果你把上述2点都验证了，确实如俺所说，就可以证明——你的“后置代理”确实通过 Tor 联网。<br/>
<br/>
　　<b>第4步——让“浏览器”通过“后置代理”联网</b><br/>
　　这一步就是配置浏览器的“代理设置”，让浏览器通过“后置代理”联网。<br/>
　　由于“Tor Browser”紧密整合了 Tor 客户端，让“Tor Browser”使用别的“后置代理”联网，可能会有点麻烦。对于“Tor Browser”的用户，俺建议：用其它浏览器，然后把“Tor Browser”只当成“Tor 客户端”来看待。<br/>
　　“后置代理”有两大类（VPN、非 VPN），俺先从【传统代理】（【非】VPN 的代理）说起，然后再说 VPN。<br/>
<br/>
　　<b>4.1——以【传统代理】作为“后置代理”</b><br/>
　　浏览器界面上的代理设置该该怎么填，取决于你使用哪个软件作为后置代理。不同类型的后置代理，浏览器中的代理设置也不同。<br/>
　　俺简单列几个常见的传统代理（非 VPN），供大伙儿参考：<br/>
<center><table border="1" cellpadding="3" cellspacing="0"><tbody>
<tr style="background:lightgrey;"><th>软件类型</th><th>【自身的】代理类型</th><th>IP 地址</th><th>端口号</th></tr>
<tr><td>赛风</td><td>HTTP 或 SOCKS</td><td>参见“注1”</td><td>参见“注2”</td></tr>
<tr><td>自由门</td><td>HTTP</td><td>参见“注1”</td><td>8580</td></tr>
<tr><td>无界</td><td>HTTP</td><td>参见“注1”</td><td>9666</td></tr>
<tr><td>蓝灯</td><td>HTTP</td><td>参见“注1”</td><td>8787</td></tr>
</tbody></table></center>
<br/>
　　注1<br/>
　　如果浏览器与“后置代理”在【同一个】系统，浏览器的“代理配置界面”中的 IP 就填写 <code>127.0.0.1</code>；如果“后置代理”在【另一个】操作系统，浏览器的“代理配置界面”中的 IP 就填写另一个操作系统的 IP。<br/>
　　注2<br/>
　　赛风同时支持 HTTP ＆ SOCKS 两种代理协议。对这两者，其“监听端口”的端口号是“随机选择”。但在它的界面上，你可以设定“本地代理端口”的端口号。你在赛风上设定的端口号是多少，在浏览器上填写的端口号也要一致。<br/>
<br/>
　　<b>4.2——以【VPN】作为“后置代理”</b><br/>
　　刚才已经介绍过【非】VPN 的情况，再来说说 VPN 做“后置代理”。<br/>
　　首先要注意的是——如果“后置代理”是 VPN，那么“后置代理”与“前置代理”【不能】在同一个系统中。因为 VPN 具有【全局代理】的效果，会影响到“前置代理”的联网。<br/>
　　接下来分两种情况——<br/>
情况1<br/>
如果“后置代理（VPN）”与浏览器在同一个系统中，浏览器【无需】做任何配置，自动就能通过 VPN 联网。<br/>
情况2<br/>
如果“后置代理（VPN）”与浏览器【不】在同一个系统中，你需要在 VPN 所在的系统中运行一个“HTTP 代理”（推荐用 Privoxy），然后把浏览器的代理设置指向 Privoxy。具体的配置教程常见如下几篇：<br/>
《<a href="../../2013/01/cross-host-use-gfw-tool.md">多台电脑如何【共享】翻墙通道——兼谈【端口转发】的几种方法</a>》<br/>
《<a href="../../2014/12/gfw-privoxy.md">如何用 Privoxy 辅助翻墙？</a>》<br/>
<br/>
<br/>
<h2>★“三重代理”的注意事项</h2>
<br/>
<h3>◇确保“上网流量”必须经过 Tor</h3>
<br/>
　　由于“三重代理”的配置方式比较复杂，某些读者可能会在配置过程中搞错。<br/>
　　如果你是因为配置错误，导致网络不通，这反而不要紧。因为你发现网络不通，自然意识到自己配置有误。<br/>
　　最危险的情况是——你配置错误，导致“上网流量【没有】经过 Tor”，此时网络依然是通的（你不会察觉到配置有误）。而你就【丧失了】Tor 网络的匿名化保护。<br/>
　　为了确保“上网流量经过 Tor”，用如下两个方式进行诊断：<br/>
<br/>
　　<b>方法1</b><br/>
　　当你上网时，观察 Tor 的流量变化。不论是用“裸 Tor”的同学，还是用“Tor Browser”的同学，都可以通过观察 Arm/Nyx 这两款工具，来判断 Tor 流量的变化。<br/>
　　关于这两款工具，本文前面章节已经说过，此处不再重复。<br/>
<br/>
　　<b>方法2</b><br/>
　　当你配置完“三重代理”之后，并且已经能通过它正常上网。然后你在上网过程中，突然【关掉 Tor 客户端】。如果上网立即中断，就说明你的“三重代理”配置确实是经过了 Tor 客户端；反之，则说明你的配置有误。<br/>
<br/>
<h3>◇【后置代理】的选择——避免客户端耍流氓</h3>
<br/>
　　从前面章节给出的两张示意图可以看出——在“三重代理”的模式下，“后置代理的客户端”可以看到你的【真实上网流量】。<br/>
　　显然，你要选一款【比较靠谱】的软件作为 Tor 的后置代理，以免该软件在你的系统中耍流氓。这里所说的“耍流氓”包含两层含义——<br/>
　　其一，<br/>
　　（你始终要记住）任何安装到你系统中的软件，（从技术上讲）都有可能偷窥你硬盘上的数据。到底会不会偷窥，就看软件作者有没有良心。<br/>
　　其二，<br/>
　　有些比较流氓的上网软件，虽然你给它设置了代理，但它【并没有】让所有的上网流量都经过代理。有些比较流氓的软件，甚至会偷偷地【尝试直连】自家的服务器，并偷偷向自家服务器上传一些用户信息。<br/>
　　如果你使用的“后置代理”存在上述行为，就会导致【Tor 被绕过】——也就是，你的“匿名性防范”出现了漏洞。<br/>
<br/>
　　为了彻底杜绝“后置代理”可能存在的“耍流氓行为”，你还需要再搭配“操作系统虚拟机”，以确保——<b>“后置代理”的【所有】流量都经过 Tor 客户端</b>。具体如何做，请看下一个章节——关于【部署方式】的讨论。<br/>
<br/>
<h3>◇【后置代理】的选择——避免暴露身份</h3>
<br/>
　　说到“避免暴露身份”，俺有必要提醒诸位——<b>【千万不要】用自己购买的 VPS 搭建 Tor 的后置代理</b>。<br/>
　　道理很简单——如果有人（比如网警）要追溯你的真实身份，通常先从【目标网站】这端开始查起。目标网站会记录【访问者 IP】，对于“三重代理”而言，这个 IP 也就是【后置代理的 IP】。如果你用自己购买的 VPS 搭建“后置代理”，不但会让 Tor 形同虚设，也会加速【身份暴露】。<br/>
<br/>
<br/>
<h2>★“三重代理”的多种【部署方式】——兼谈优缺点对比</h2>
<br/>
　　对于“三重代理”这个招数，牵涉的网络模块有4个——前置代理、Tor、后置代理、浏览器或网络软件。<br/>
　　这4个东东，既可以全部放在一个系统，也可以分别部署到两个甚至更多系统。排列组合，有很多种部署方式。<br/>
　　篇幅有限，俺只挑选几种比较典型的部署方式，略作点评。<br/>
　　在本章节的如下几个小节中，凡是提及“OS”，既可以是“物理系统”（Host OS），也可以是“虚拟系统”（Guest OS）。<br/>
　　下面讨论的几种部署方式，都是拿“浏览器”来说事儿；但如果把“浏览器”换成“其它网络软件”，道理也一样。<br/>
<br/>
<h3>◇部署方式1：ALL in ONE</h3>
<br/>
　　最简单的，当然是全部放到同一个 OS 里面。但这种方式，安全隐患也最大。至少有如下几个缺点/风险。<br/>
<br/>
　　<b>缺点1：</b><br/>
　　由于“前置代理”在这个 OS 中，这个 OS 必然是可以直接连【外网】。因此，“后置代理”以及“浏览器的插件”有可能绕过 Tor 直接联网——那“匿名网络”就形同虚设。<br/>
　　<b>缺点2：</b><br/>
　　因为所有都放在一起，如果“前置代理 or 后置代理”耍流氓，有可能偷窥到浏览器的敏感数据（比如：浏览器的 cookie 中会保存网站登录令牌）<br/>
　　<b>缺点3：</b><br/>
　　如前面章节所说——当“前置代理 ＆ 后置代理”放在同一个系统，“后置代理”【不能】选择 VPN。<br/>
<br/>
<h3>◇部署方式2：单独隔离浏览器</h3>
<br/>
　　所谓的“单独隔离浏览器”就是——浏览器单独放一个系统（隔离 OS），另外3个东东放一个系统（网关 OS）。<br/>
　　如此一来，“前置代理 ＆ 后置代理”都【无法】偷窥浏览器的敏感数据。<br/>
<br/>
　　<b>缺点1：</b><br/>
　　由于“前置代理 ＆ 后置代理”放在同一个 OS 中，这个 OS 必然是可以直接连【外网】。因此，“后置代理”有可能绕过 Tor 直接联网——那“匿名网络”就形同虚设。<br/>
　　<b>缺点2：</b><br/>
　　如前面章节所说——当“前置代理 ＆ 后置代理”放在同一个系统，“后置代理”【不能】选择 VPN。<br/>
<br/>
<h3>◇部署方式3：“前置代理＆Tor” ＋ “后置代理＆浏览器”</h3>
<br/>
　　这种方式把“前置代理＆Tor”放在一个 OS（网关 OS），“后置代理＆浏览器”放另一个 OS（隔离 OS）。<br/>
　　然后再设置“隔离 OS 的防火墙”，使它只能访问“网关 OS 的 Tor 端口”。如此一来，“后置代理＆浏览器”这俩都不可能绕过 Tor 直接联网。<br/>
<br/>
　　<b>缺点1：</b><br/>
　　因为“后置代理＆浏览器”放在同一个 OS，如果“后置代理”耍流氓，有可能偷窥到浏览器的敏感数据（比如：浏览器的 cookie 中会保存网站登录令牌）<br/>
<br/>
<h3>◇部署方式4：“前置代理＆Tor” ＋ “后置代理” ＋ “浏览器”</h3>
<br/>
　　这种部署方式需要【3个】OS，放“前置代理＆Tor”的是“网关 OS”，另外两个是“隔离 OS”。为了安全起见，你可以通过配置各个 OS 的防火墙，使得“浏览器的 OS”只能访问“后置代理的 OS 的特定端口”，而“后置代理的 OS”只能访问“网关 OS 的 Tor 端口”。<br/>
　　如此一来，基本上能解决前面几种部署方式的各种缺点。<br/>
　　虽然这种部署比前几种都要好，但完美的方案是不可能滴！这种部署方式的缺点是——需要【3个】OS，增加了硬件开销。当然啦，如果你很看重安全性，增加一点硬件开销也没啥！<br/>
<br/>
<h3>◇关于“部署方式”的建议</h3>
<br/>
　　如果你对网络配置不是特别熟悉，建议先从“部署方式1”（ALL in ONE）开始入手。因为这种【单系统】的部署方式最简单。<br/>
　　等你把“单系统”的部署方式搞定了，再慢慢尝试“多系统”的部署方式。<br/>
<br/>
<br/>
<b>俺博客上，和本文相关的帖子（需翻墙）</b>：<br/>
《<a href="../../2013/11/tor-faq.md">“如何翻墙”系列：关于 Tor 的常见问题解答</a>》<br/>
《<a href="../../2009/09/break-through-gfw-with-tor.md">戴“套”翻墻的方法</a>》<br/>
《<a href="../../2018/04/gfw-tor-browser-7.5-meek.md">“如何翻墙”系列：扫盲 Tor Browser 7.5——关于 meek 插件的配置、优化、原理</a>》<br/>
《<a href="../../2015/03/Tor-Arm.md">扫盲 Arm——Tor 的界面前端（替代已死亡的 Vidalia）</a>》<br/>
《<a href="../../2010/04/howto-cover-your-tracks-0.md">如何隐藏你的踪迹，避免跨省追捕</a>》（系列）<br/>
《<a href="../../2012/10/system-vm-0.md">扫盲操作系统虚拟机</a>》（系列）<br/>
《<a href="../../2013/01/cross-host-use-gfw-tool.md">多台电脑如何【共享】翻墙通道——兼谈【端口转发】的几种方法</a>》<br/>
《<a href="../../2014/12/gfw-privoxy.md">如何用 Privoxy 辅助翻墙？</a>》<br/>
《<a href="../../2019/04/Proxy-Tricks.md">如何让【不支持】代理的网络软件，通过代理进行联网（不同平台的 N 种方法）</a>》
</div>


------------------------------------------------

版权声明本博客所有的原创文章，作者皆保留版权。转载必须包含本声明，保持本文完整，并以超链接形式注明作者编程随想和本文原始地址：https://program-think.blogspot.com/2020/08/Tor-Triple-Proxy.html