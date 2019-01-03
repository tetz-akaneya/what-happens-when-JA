...のとき何がおきているのか
====================

このレポジトリは昔からある「google.comとブラウザのアドレスバーに打ちこんでEnterを押すとき一体何が起きているのか？」という問いに答えるのものです。

通常の回答と異なり、全ての部分について可能な限り深く答えます。

この作業はみんなで行なっています。是非参加してください！
まだまだ詳細を説明し切れていない部分がたくさんあります。
よければ追記をお願いします！プルリクエストを送ってください、お願いします！

ライセンスは全てクリエイティブ・コモンズゼロに従います。

中国語で読みたい方は `简体中文` から、韓国語は `한국어` からどうぞ。注: これらはalex/what-happens-whenのレポジトリのメンテナが内容をチェックしたわけではありません。

Table of Contents
====================

.. contents::
   :backlinks: none
   :local:

"g"キーが押されると
----------------------
キーボードの物理的な動作とOSの割り込みについてです。
"g"キーが押されるとブラウザはイベントを受けとり自動補間を行います。
ブラウザのURLバーの下に、各ブラウザのアルゴリズムやプライベート/インコグニッションなどのモードによって異なる様々な候補がURLバーの下にドロップボックスとして表示されます。
こういったアルゴリズムのほとんどは検索履歴、ブックマーク、クッキー、またインターネット全体での検索ワードの人気を利用して、結果の並び替えや優先順序付けを行います。
"google.com"と入力するにつれたくさんのコードが実行されていき、1つのキーを打つごとに候補が正確になっていきます。
入力が終わる前に"google.com"を候補にあげてくれることさえあるかもしれません。

はじめは"Enter"キー
---------------------------

キーボードのEnterキーを押すところから始めましょう。

このタイミングででEnterキーの電気回路が（物理的にあるいは電気容量的に）閉じます。

これによりキーボードの論理回路にわずかな電流が流れます。この論理回路は各キーのスイッチの状態を網羅的に調べ、素早く開閉するスイッチによる電気的ノイズをの除き、キーコードの整数、今回の場合13に変換します。

次にキーボードのコントローラーがキーコードをコンピュータに伝えるためエンコードします。
これは現在ではほぼ必ずUniversal Serial Bus (USB)またはBluetooth connectionにより行われますが、歴史的にはPS/2またはADBコネクションにより行われてきました。

USBキーボードの場合

The USB circuitry of the keyboard is powered by the 5V supply provided over pin 1 from the computer's USB host controller.
キーボードのUSB回路はコンピュータのUSBホストコントローラ(訳注: キーボード側にUSBデバイスコントローラがあり、対でUSBが機能する)のpin1によって供給される5Vの電気で動いています。

生成されたキーコードは、"endpoint"と呼ばれるキーボードのレジスタの回路記憶に保存されます。

USBホストコントローラはその"endpoint"をおよそ10m秒ごと（キーボードにより指定された最小の値）にポーリングし、保存されたキーコードの値を取得します。

この値はUSB SIE(Serial Interface Engine)へ伝わり、1つあるいは複数のUSBパケットに変換されます。ついでUSBの低レイヤープロトコルがこのパケットを処理します。

このパケットはD+およびD-ピン(真ん中の2つ)の電気シグナルにより最大スピード(1.5Mb/s)で送信されます。
なので、いつもHID (Human Interface Device)はUSB 2.0のコンプライアンスにより"low speed devise"と言われています。

この連番シグナルはコンピュータのUSBホストコントローラでデコードされ、Human Interface Device (HID)のキーボードデバイスドライバーによって解釈されます。

ついでキーの値はOSのハードウェア抽象化レイヤーに渡されます。

仮想キーボードの場合(タッチスクリーンなど)

When the user puts their finger on a modern capacitive touch screen, a tiny amount of current gets transferred to the finger.
ユーザが指で電気容量型のタッチスクリーンに触れると、ほんの小さな量の電流が指に流れます。

This completes the circuit through the electrostatic field of the conductive layer and creates a voltage drop at that point on the screen.
この電流と導電層の静電場により回路が完成し、そのスクリーンのその点に電圧降下が生じます。

The screen controller then raises an interrupt reporting the coordinate of the key press.
スクリーンコントローラは割り込みを発生させ、キーが押された軸を伝えます。

Then the mobile OS notifies the current focused application of a press event in one of its GUI elements (which now is the virtual keyboard application buttons).
次にモバイルOSは現在開かれているアプリのGUI要素のどれかにプレスイベントを発生させます（今回の場合は仮想キーボードですが）。

The virtual keyboard can now raise a software interrupt for sending a 'key pressed' message back to the OS.
この仮想キーボードは割り込みソフトウェア割り込みを発生させ、「キーが押された」というメッセージをOSに伝えます。

This interrupt notifies the current focused application of a 'key pressed' event.
この割り込みが現在開かれているアプリに「キーが押された」というイベントを知らせます。

Interrupt fires [NOT for USB keyboards]
割り込み発火（USBキーボード以外）

The keyboard sends signals on its interrupt request line (IRQ), which is mapped to an interrupt vector (integer) by the interrupt controller.
キーボードはシグナルをinterrupt request(IRQ:割り込み要求)を通じて送ります。IRQは割り込みコントローラによって割り込みベクタ(整数)にマッピングされています。

The CPU uses the Interrupt Descriptor Table (IDT) to map the interrupt vectors to functions (interrupt handlers) which are supplied by the kernel. 
CPUは割り込みデスクリプタテーブル(Interrupt Descriptor Table:IDT)を利用して割り込みベクタをカーネルから提供される各機能（割り込みハンドラ）にマッピングします。

When an interrupt arrives, the CPU indexes the IDT with the interrupt vector and runs the appropriate handler. Thus, the kernel is entered.
割り込みが起きると、CPUはIDTを割り込みベクタでインデクスし、対応するハンドラを実行します。カーネルにたどり着きました。

(On Windows) A WM_KEYDOWN message is sent to the app
(ウインドウズの場合)WM_KEYDOWNメッセージがアプリに送られます。
The HID transport passes the key down event to the KBDHID.sys driver which converts the HID usage into a scancode.
HIDトランスポートはkey downイベントをKBDHID.sysドライバに伝え、ついでKBDHID.sysドライバはHID usageをスキャンコードに変換します。

In this case the scan code is VK_RETURN (0x0D). The KBDHID.sys driver interfaces with the KBDCLASS.sys (keyboard class driver).
この場合スキャンコードはVK_RETURN(0x0D)です。KBDHID.sysドライバはKBDCLASS.sys(キーボードクラスドライバ)とやりとりを行います。

This driver is responsible for handling all keyboard and keypad input in a secure manner.
このドライバは安全に全てのキーボードおよびキーパッドの入力を処理する責任があります。

It then calls into Win32K.sys (after potentially passing the message through 3rd party keyboard filters that are installed). This all happens in kernel mode.
ついでWin32K.sys（もしかすると外部からインストールされたサードバーティ製ののキーボードフィルタを経たあと）が動作します。
これは全てカーネルモードで起こります。

Win32K.sys figures out what window is the active window through the GetForegroundWindow() API.
Win32K.sysはGetForegroundWindow()APIを用いてどのウィンドウがアクティブかを判断します。

This API provides the window handle of the browser's address box.
このAPIによりブラウザのアドレスボックスのWindowハンドルが得られます。

The main Windows "message pump" then calls SendMessage(hWnd, WM_KEYDOWN, VK_RETURN, lParam).
ついでWindowsの"message pump"がSendMessage(hWnd, WM_KEYDOWN, VK_RETURN, lParam)を呼びます。

lParam is a bitmask that indicates further information about the keypress: repeat count (0 in this case), the actual scan code (can be OEM dependent, but generally wouldn't be for VK_RETURN), whether extended keys (e.g. alt, shift, ctrl) were also pressed (they weren't), and some other state.
lParamはキーの押下に関するさらなる情報を示すビットマスクです。情報とはすなわちリピート回数(今回の場合は0)や実際のスキャンコード(OEMに依存しているかも知れませんが、一般にVK_RETURNの場合はOEM依存ではありません)、また他のalt, shift, ctrlなどが一緒に押されてたか、などの情報です。

The Windows SendMessage API is a straightforward function that adds the message to a queue for the particular window handle (hWnd).
WindowsのSendMessage APIは特定のWindowハンドル(hWnd)に対するキューにそのメッセージを追加する分かりやすい機能です。

Later, the main message processing function (called a WindowProc) assigned to the hWnd is called in order to process each message in the queue.
hWndに割り当てられたWindowProcと呼ばれるメインのメッセージ処理機能が呼ばれて、キューに入ったメッセージは処理されていきます。

The window (hWnd) that is active is actually an edit control and the WindowProc in this case has a message handler for WM_KEYDOWN messages.
そのアクティブなwindow(hWnd)は実はエディットコントロールで、WindowProcはこの場合WM_KEYDOWNメッセージのためのメッセージハンドラを持ちます。

This code looks within the 3rd parameter that was passed to SendMessage (wParam) and, because it is VK_RETURN knows the user has hit the ENTER key.
このコードはSendMessaタイミングで (wParam)に渡された3番目の引見ます。今回はEnterキーを押している)VK_RETURNなのーザがエンタキーあるいはを押したことが分かります。

(On OS X) A KeyDown NSEvent is sent to the app
(OS Xの場合)KeyDown NSEventがアプリに送られる

The interrupt signal triggers an interrupt event in the I/O Kit kext keyboard driver.
割り込みシグナルがI/O Kit kextキーボードドライバに割り込みイベントを発生させます。

The driver translates the signal into a key code which is passed to the OS X WindowServer process.
このドライバは受け取ったシグナルをキーコードに変換してOS X WindowServerプロススに渡します。

Resultantly, the WindowServer dispatches an event to any appropriate (e.g. active or listening) applications through their Mach port where it is placed into an event queue.
最終的にWindowServerは適切な(例えばアクティブまたはリスニング状態の)アプリにMachポート経由でイベントをdispatchします。イベントはポートのイベントキューに入ります。

Events can then be read from this queue by threads with sufficient privileges calling the mach_ipc_dispatch function.
イベントはmach_ipc_dispatchを実行できるだけの権限をもつスレッドによって読み込まれます。

This most commonly occurs through, and is handled by, an NSApplication main event loop, via an NSEvent of NSEventType KeyDown.
これは、NSEvent of NSEventType KeyDown経由のNSApplicationメインイベントループにより最もよく起き、処理されます。

(On GNU/Linux) the Xorg server listens for keycodes
(GNU/Linuxの場合)Xorgサーバがキーコードをlistenする

When a graphical X server is used, X will use the generic event driver evdev to acquire the keypress.
グラフィカルXサーバを利用する場合。Xサーバはキーを入力を得るためgeneric event driver(evdev)を利用します。

A re-mapping of keycodes to scancodes is made with X server specific keymaps and rules.
キーコードからスキャンコードへのリマッピングはXサーバ特有のキーマップとルールで行われます。

When the scancode mapping of the key pressed is complete, the X server sends the character to the window manager (DWM, metacity, i3, etc), so the window manager in turn sends the character to the focused window.
押されたキーからスキャンコードへのマッピングが終わると、Xサーバはその文字をウィンドウマネジャー(DWM, metacity, i3など)へ送ります。
逆にウィンドウマネジャーは文字を対象のウィンドウへ送ります。

The graphical API of the window that receives the character prints the appropriate font symbol in the appropriate focused field.
その文字を受け取ったウィンドウのグラフィカルAPIは、適切な文字を適切なフィールドに表示します。

Parse URL
URLをパースする
The browser now has the following information contained in the URL (Uniform Resource Locator):
ブラウザはURL(Uniform Resource Locator)から次の情報を得ることができます。

Protocol "http"
Use 'Hyper Text Transfer Protocol'
プロトコルは"Hyper Text Transfer Protocol"を使う
Resource "/"
Retrieve main (index) page
メインページを取りに行く
Is it a URL or a search term?
これはURLか？検索単語か？
When no protocol or valid domain name is given the browser proceeds to feed the text given in the address box to the browser's default web search engine.
プロトコルがない、あるいは有効なドメインでない場合、ブラウザは入力されたテキストをブラウザのデフォルトの検索エンジンに渡します。

In many cases the URL has a special piece of text appended to it to tell the search engine that it came from a particular browser's URL bar.
多くの場合、URLは特別なテキストが追加されるので、サーチエンジンはそのテキストがどのブラウザのURLバーから来たものなのかを知ることができます。

Convert non-ASCII Unicode characters in hostname
ホストネームに含まれるASCIIユニコード文字でない文字を変換する

The browser checks the hostname for characters that are not in a-z, A-Z, 0-9, -, or ..
ブラウザはホストネームの文字の中に「a-z, A-Z, 0-9, -, .」以外の文字がないか調べます。

Since the hostname is google.com there won't be any, but if there were the browser would apply Punycode encoding to the hostname portion of the URL.
Check HSTS list
今回の場合ホストネームは"google.com"なのでそういった文字はありませんが、もしある場合にはURLのホストネーム部分にPunycodeエンコーディングを適用します。

The browser checks its "preloaded HSTS (HTTP Strict Transport Security)" list.
ブラウザは"preloaded HSTS(HTTP Strict Transport Security)"リストを調べます。

This is a list of websites that have requested to be contacted via HTTPS only.
これはHTTPSでのみリクエストを送るように求めているウェブサイトの一覧です。

If the website is in the list, the browser sends its request via HTTPS instead of HTTP.
もしそのウェブサイトがリストにあれば、ブラウザはHTTPではなくHTTPでリクエストを送ります。

Otherwise, the initial request is sent via HTTP.
なければ最初のリクエストはHTTPで送られます。

(Note that a website can still use the HSTS policy without being in the HSTS list.
ウェブサイトは、HSTS一覧になくてもHSTSポリシーを利用可能であることに注意してください。

The first HTTP request to the website by a user will receive a response requesting that the user only send HTTPS requests.
最初のHTTPリクエストに対するレスポンスは、HTTPSリクエストのみでリクエストを送ることを要求するものです。

However, this single HTTP request could potentially leave the user vulnerable to a downgrade attack, which is why the HSTS list is included in modern web browsers.)
しかし、この1回のHTTPリクエストによりユーザはダウングレード攻撃を受ける可能性があります。そのため、現在のWebブラウザにはHSTS一覧が搭載されています。


DNS lookup
DNSルックアップ

Browser checks if the domain is in its cache. (to see the DNS Cache in Chrome, go to chrome://net-internals/#dns).
ブラウザは対象のドメインがキャッシュにないか調べます。(ChromeのDNSキャッシュを見たければ、chrome://net-internals/#dnsにアクセスしてください)

If not found, the browser calls gethostbyname library function (varies by OS) to do the lookup.
もしキャッシュになければ、ブラウザはgethostbynameライブラリ関数(OSにより異なる)を呼んで、ルックアップを行います。

gethostbyname checks if the hostname can be resolved by reference in the local hosts file (whose location varies by OS) before trying to resolve the hostname through DNS.
gethostbynameはホストネームの名前解決をするのに、DNSによる名前解決の前にローカルのホストファイル(OSにより場所は異なる)で解決できるか確認します。

If gethostbyname does not have it cached nor can find it in the hosts file then it makes a request to the DNS server configured in the network stack.
gethostbynameがキャッシュに持っていなかったりホストファイルにない場合は、ネットワークスタックで設定されたネットワークDNSサーバにリクエストを送ります。

This is typically the local router or the ISP's caching DNS server.
典型的なのは、ローカルのルーターかISPのキャッシュDNSサーバです。

If the DNS server is on the same subnet the network library follows the ARP process below for the DNS server.
もしDNSサーバが同じサブネットにあれば、ネットワークライブラリはそのDNSサーバに対するARP処理に従います。

If the DNS server is on a different subnet, the network library follows the ARP process below for the default gateway IP.
もしDNSサーバが異なるサブネットにあれば、ネットワークライブラリはデフォルトゲートウェイIPに対するARP処理に従います。

ARP処理

In order to send an ARP (Address Resolution Protocol) broadcast the network stack library needs the target IP address to look up.
ARP(Address Resolution Protocol)ブロードキャストを行うため、ネットワークスタックライブラリは対象のIPアドレスを知る必要があります。

It also needs to know the MAC address of the interface it will use to send out the ARP broadcast.
また、ARPブロードキャストを行うため、MACアドレスを知る必要もあります。

The ARP cache is first checked for an ARP entry for our target IP.
ARPキャッシュにARPエントリのターゲットIPがないか調べます。

If it is in the cache, the library function returns the result: Target IP = MAC.
キャッシュにあれば、ライブラリは次のような結果を返します: Target IP = MAC

If the entry is not in the ARP cache:
もしエントリーがARPキャッシュにない場合

* The route table is looked up, to see if the Target IP address is on any of
  the subnets on the local route table. If it is, the library uses the
  interface associated with that subnet. If it is not, the library uses the
  interface that has the subnet of our default gateway.

* ターゲットIPアドレスがローカルのルートテーブルのサブネットのいずれかにないかが調べられます。もしあればライブラリはそのサブネットのインターフェースを利用します。もしなければ、ライブラリはデフォルトゲートウェイのサブネットのインターフェースを利用します。

* The MAC address of the selected network interface is looked up.
* 選択したネットワークインタフェースのMACアドレスを調べます。

* The network library sends a Layer 2 (data link layer of the `OSI model`_)
  ARP request:
* ネットワークライブラリはLayer2(OSIモデルにおけるデータリンク層)にARPリクエストを送ります。

``ARPリクエスト``::
    送信者MAC: interface:mac:address:here
    送信者IP: interface.ip.goes.here
    ターゲット MAC: FF:FF:FF:FF:FF:FF (Broadcast)
    ターゲット IP: target.ip.goes.here

コンピュータとルータの間にあるハードウェアの種類によって以下のように変化します。

直接繋がれている場合:

* コンピュータがルータと直接接続されている場合、ルータはARPリプライを返します。

ハブの場合

* コンピュータがハブに繋がっている場合、ハブはARPリクエストを他の全てのポートにブロードキャストします。もしルータが同じワイヤに繋がっている場合、ルータはARPリプライを返します。

スイッチの場合

* コンピュータがスイッチに繋がっている場合、スイッチはローカルのCAM/MACテーブルからどのポートが探しているMACアドレスを持っているのか調べます。もしそのMACアドレスに対するエントリがなければ、他の全てのポートへARPリクエストをブロードキャストします。

* また、もしスイッチのMAC/CAMテーブルにそのMACアドレスがあれば、ARPリクエストをそのポートに送ります。
* また、もしルータが同じワイヤ上にあれば、ARPリプライを返します。

``ARP Reply``::

    送信者MAC: target:mac:address:here
    送信者IP: target.ip.goes.here
    ターゲットMAC: interface:mac:address:here
    ターゲットIP: interface.ip.goes.here

Now that the network library has the IP address of either our DNS server or
the default gateway it can resume its DNS process:
ネットワークライブラリが自分たちのDNSサーバあるいはデフォルトゲートウェイのIPアドレスを持っているので、DNSの処理を進めることができます。

* Port 53 is opened to send a UDP request to DNS server (if the response size
  is too large, TCP will be used instead).

* 53番ポートが開いて、DNSサーバにUDPリクエストを送ります(レスポンスサイズが大きすぎる場合は代わりにTCPが利用されます)。
* If the local/ISP DNS server does not have it, then a recursive search is
  requested and that flows up the list of DNS servers until the SOA is reached,
  and if found an answer is returned.
* もしローカルまたはISPのDNSサーバがIPを知らなければ、再帰的探索がリクエストされて、一連のDNSサーバをたどり、SOAにたどり着き、もしあればAnswerが返されます。

Opening of a socket
ソケットを開く
-------------------
Once the browser receives the IP address of the destinatio server, it takes
that and the given port number from the URL (the HTTP protocol defaults to port
80, and HTTPS to port 443), and makes a call to the system library function
named ``socket`` and requests a TCP socket stream - ``AF_INET/AF_INET6`` and
``SOCK_STREAM``.

ブラウザが目標サーバのIPを受け取ると、それとURLから得た適切なポート(HTTPは80, HTTPSは443)を用いてsocketという名前のシステム関数を呼び、TCPソケットストリーム(``AF_INET/AF_INET6`` と
``SOCK_STREAM``)をリクエストします。

* This request is first passed to the Transport Layer where a TCP segment is
  crafted. The destination port is added to the header, and a source port is
  chosen from within the kernel's dynamic port range (ip_local_port_range in
  Linux).

* このリクエストははじめにTCPセグメントが生成されるトランスポートレイヤに渡されます。標的ポートがヘッダに追加され、ソースポートがカーネルの動的ポート幅(Linuxではip_local_port_range)から選ばれます。

* This segment is sent to the Network Layer, which wraps an additional IP
  header. The IP address of the destination server as well as that of the
  current machine is inserted to form a packet.

* このセグメントはネットワークレイヤに送られIPヘッダが付与されます。標的サーバおよびクライアントののIPアドレスを利用してパケットが作られます。

* The packet next arrives at the Link Layer. A frame header is added that
  includes the MAC address of the machine's NIC as well as the MAC address of
  the gateway (local router). As before, if the kernel does not know the MAC
  address of the gateway, it must broadcast an ARP query to find it.

* パケットはついでリンクレイヤに到着します。MACアドレスのゲートウェイ(ローカルルータ)およびNICのMacアドレスを含むフレームヘッダが付与されます。前と同じように、もしカーネルがゲートウェイのMACアドレスを知らない場合ARPリクエストを行なって探します。

At this point the packet is ready to be transmitted through either:
この時点でパケットは既にeitherを通じてやりとりされています。

* `Ethernet`_
* `WiFi`_
* `Cellular data network`_

For most home or small business Internet connections the packet will pass from
your computer, possibly through a local network, and then through a modem
(MOdulator/DEModulator) which converts digital 1's and 0's into an analog
signal suitable for transmission over telephone, cable, or wireless telephony
connections. On the other end of the connection is another modem which converts
the analog signal back into digital data to be processed by the next `network
node`_ where the from and to addresses would be analyzed further.

ほとんどの家庭用、あるいは小さなビジネス用のインターネットにおいてパケットはあなたのコンピュータから、場合によってはローカルネットワークを経由して、モデム(MOdulator/DEModulator)を通り、1と0のデジタルな情報を電話やケーブル、その他ワイヤレスな通信に適したアナログな形に変換します。コネクションの反対側では、別のモデムがそのアナログなデータをデジタルなデータに変換し、次の`network node`に渡されます。ネットワークノードでは送信者および受信者のアドレスがより詳細に解析されます。

Most larger businesses and some newer residential connections will have fiber
or direct Ethernet connections in which case the data remains digital and
is passed directly to the next `network node`_ for processing.

また大きな会社のほとんど、また新しい住宅のいくつかはファイバーかEthernetに直接つながっており、この場合データはデジタルのまま直接次の`network node`へと渡されます。


Eventually, the packet will reach the router managing the local subnet. From there, it will continue to travel to the autonomous system's (AS) border routers, other ASes, and finally to the destination server. Each router along the way extracts the destination address from the IP header and routes it to the appropriate next hop. The time to live (TTL) field in the IP header is decremented by one for each router that passes. The packet will be dropped if the TTL field reaches zero or if the current router has no space in its queue (perhaps due to network congestion).

そしてパケットはローカルサブネットを管理するルーターにたどり着きます。ここから、AS(autonomous system's)ボーダールーターや他のASに行き、最終的に標的のサーバにたどり着きます。移動経路上にあった各ルータはIPヘッダから標的サーバのアドレスを読み取り、適切な次のルータへと導きます。IPヘッダのTTL(time to live)フィールドはルータを1つ経るごとに1減ります。パケットはTTLが0に到達するか現在のルータのキューにスペースがないと、破棄されます。


This send and receive happens multiple times following the TCP connection flow:
この送受信は以下のTCPコネクションの流れの中で何回か行われます。

* Client chooses an initial sequence number (ISN) and sends the packet to the server with the SYN bit set to indicate it is setting the ISN
* クライアントはISN(initial sequence number : 初期連番番号)を決め、SYNビットをセットしてISNを設定しようとしていることを表しつつパケットをサーバに送ります。

* Server receives SYN and if it's in an agreeable mood:
* サーバはSYNを受け取ります。もし受け取り可能な場合、
   * Server chooses its own initial sequence number
   * サーバは自身でISNを決めます。
   * Server sets SYN to indicate it is choosing its ISN
   * サーバはISNを選択しようとしていることを伝えるため、SYNをセットします。
   * Server copies the (client ISN +1) to its ACK field and adds the ACK flag to indicate it is acknowledging receipt of the first packet
   * サーバはクライアントのISN+1の値を計算し、ACKフィールドに設定します。またACKフラグを設定して最初のパケットのリクエストを承認します。
* Client acknowledges the connection by sending a packet:
* クライアントは以下のようなパケットを送ることでコネクションを承認します。
   * Increases its own sequence number
   * 自身のシーケンス番号を増やす
   * Increases the receiver acknowledgment number
   * 受信者側のACK番号を増やす
   * Sets ACK field
   * ACKフィールドを設定する
* Data is transferred as follows:
* データは以下のように通信されます
   * As one side sends N data bytes, it increases its SEQ by that number
   * 片側がNバイトのデータを送ると、SEQをその番号分増やします。
   * When the other side acknowledges receipt of that packet (or a string of packets), it sends an ACK packet with the ACK value equal to the last received sequence from the other
   * もう片側が
* To close the connection:
   * The closer sends a FIN packet
   * The other sides ACKs the FIN packet and sends its own FIN
   * The closer acknowledges the other side's FIN with an ACK

TLSハンドシェイク
-------------
* The client computer sends a ``ClientHello`` message to the server with its
  Transport Layer Security (TLS) version, list of cipher algorithms and
  compression methods available.
* クライアントがClientHelloメッセージをTLSバージョン、可能な暗号化アルゴリズムおよび圧縮方法のリストと共にサーバに送ります。

* The server replies with a ``ServerHello`` message to the client with the
  TLS version, selected cipher, selected compression methods and the server's
  public certificate signed by a CA (Certificate Authority). The certificate
  contains a public key that will be used by the client to encrypt the rest of
  the handshake until a symmetric key can be agreed upon.

* サーバはTLSのバージョン、選択した暗号化アルゴリズムおよび圧縮方法、CA(Certificate Authorityより署名された)サーバーの公開証明書と共に、ServerHelloメッセージでレスポンスを返します。

* The client verifies the server digital certificate against its list of
  trusted CAs. If trust can be established based on the CA, the client
  generates a string of pseudo-random bytes and encrypts this with the server's
  public key. These random bytes can be used to determine the symmetric key.

* クライアントはサーバの電子証明書を、信用しているCAのリストに照会します。サーバのCAが信用できるとなった場合、クライアントは擬似ランダムな文字列を生成してこれをサーバの公開鍵で暗号化します。このランダムな文字列は共通鍵として利用されます。

* The server decrypts the random bytes using its private key and uses these
  bytes to generate its own copy of the symmetric master key.
* サーバはプライベートキーで受け取ったランダム文字列を復号して、共通鍵を取得します。

* The client sends a ``Finished`` message to the server, encrypting a hash of
  the transmission up to this point with the symmetric key.
* クライアントはここまでにあったやりとりのハッシュ値を公開鍵で暗号化して、``Finished``メッセージをサーバに送ります。

* The server generates its own hash, and then decrypts the client-sent hash
  to verify that it matches. If it does, it sends its own ``Finished`` message
  to the client, also encrypted with the symmetric key.
* サーバは自身でもハッシュを生成し、クライアントから送られてきたハッシュ値と比較します。もしあっていれば、サーバからも共通鍵で暗号化したFinishedメッセージをクライアントに送ります。

* From now on the TLS session transmits the application (HTTP) data encrypted
  with the agreed symmetric key.
* これ以降は、TLSセッションによりアプリケーションのデータは共通鍵で暗号化されてやりとりされます。

HTTPプロトコル
-------------

もし利用しているウェブブラウザがGoogle製なら、ページを取得にはHTTPリクエストを送る代わりにHTTPからSPDYプロトコルにアップグレードするようなリクエストを送ります。

If the client is using the HTTP protocol and does not support SPDY, it sends a
request to the server of the form::

クライアントがHTTPプロトコルを使っていてかつSPDYをサポートしていない場合、ブラウザは以下の以下の形式で送ります。

    GET / HTTP/1.1
    Host: google.com
    Connection: close
    [other headers]

``[other headers]``はHTTP規約で定められた、いくつかのキーと値のペアで、ペア同士は改行で区切られます。(これはブラウザがHTTP規約を守り、HTTP/1.1を利用している場合に限ります。もしそうでければリクエストにHostヘッダーもないかもしれず、この場合バージョンはHTTP/1.0かHTTP/0.9が利用されます)
where ``[other headers]`` refers to a series of colon-separated key-value pairs
formatted as per the HTTP specification and separated by single new lines.
(This assumes the web browser being used doesn't have any bugs violating the
HTTP spec. This also assumes that the web browser is using ``HTTP/1.1``,
otherwise it may not include the ``Host`` header in the request and the version
specified in the ``GET`` request will either be ``HTTP/1.0`` or ``HTTP/0.9``.)

HTTP/1.1は送信者が"close"Connectionオプションをつけることができます。これをつけるとコネクションはレスポンスが返った後に閉じることを示唆します。例えば、

HTTP/1.1 defines the "close" connection option for the sender to signal that
the connection will be closed after completion of the response. For example,

    Connection: close

のようなものです。
接続を維持する機能をサポートしていないHTTP/1.1アプリケーションは必ず"close"コネクションオプションを全てのメッセージに含める必要があります。
HTTP/1.1 applications that do not support persistent connections MUST include
the "close" connection option in every message.

リクエストとヘッダーを送った後はブラウザは改行文字1つだけを送り、サーバ側にリクエストが終わったことを伝えます。

After sending the request and headers, the web browser sends a single blank
newline to the server indicating that the content of the request is done.

サーバはリクエストの結果を表すレスポンスコードなどを以下のようなフォーマットで返します。

The server responds with a response code denoting the status of the request and responds with a response of the form::

    200 OK
    [レスポンス ヘッダ]

この次の改行文字のあと、www.google.comのHTMLが続きます。次にサーバはコネクションを切るか、あるいはクライアントのリクエストヘッダによってはつなぎ続けてさらなるリクエストを待ちます。

Followed by a single newline, and then sends a payload of the HTML content of
``www.google.com``. The server may then either close the connection, or if
headers sent by the client requested it, keep the connection open to be reused
for further requests.

ブラウザから送信されたHTTPヘッダから、ブラウザのファイルのキャッシュバージョン(ETagヘッダなど)を見て、最後に取得した時から変更がないことにサーバが気づいた場合、次のようなレスポンスを返すこともあります。

If the HTTP headers sent by the web browser included sufficient information for
the web server to determine if the version of the file cached by the web
browser has been unmodified since the last retrieval (ie. if the web browser
included an ``ETag`` header), it may instead respond with a request of
the form::

    304 Not Modified
    [レスポンス ヘッダ]

and no payload, and the web browser instead retrieves the HTML from its cache.

それ以外の内容はなく、ブラウザはキャッシュからHTMLを取得することになります。

After parsing the HTML, the web browser (and server) repeats this process
for every resource (image, CSS, favicon.ico, etc) referenced by the HTML page,
except instead of ``GET / HTTP/1.1`` the request will be
``GET /$(URL relative to www.google.com) HTTP/1.1``.

HTMLのパース後、ウェブブラウザ(およびサーバ)はこの処理をHTMLページから参照されるリソース(画像、CSS、ファビコンなど)ごとに繰り返します。


If the HTML referenced a resource on a different domain than
``www.google.com``, the web browser goes back to the steps involved in
resolving the other domain, and follows all steps up to this point for that
domain. The ``Host`` header in the request will be set to the appropriate
server name instead of ``google.com``.

HTMLがwww.google.comと異なるドメインのリソースを参照していた場合、ウェブブラウザはそのドメインを名前解決するところまで戻ってそこから再開します。リクエストのHostヘッダはgoogle.comでなく別の適切な名前に設定されます。


HTTPサーバリクエスト処理
--------------------------
サーバサイド側でリクエスト/レスポンスを処理しているのはHTTPD(HTTPデーモン)サーバです。1番一般的なHTTPDサーバはリナックスの場合Apacheかnginxで、Windowsの場合はIISです。

* HTTPDがリクエストを受け取ります。
* The HTTPD (HTTP Daemon) receives the request.
* サーバはリクエストを分解して以下のパラメタをチェックします。
  * HTTPリクエストメソッド(GET, HEAD, POST, PUT, DELETE, CONNECT, OPTIONS, TRACE)。URLバーに直接打ち込んだ今回の場合、このパラメタはGETになります。
  * ドメイン。今回の場合はgoogle.com
  * リクエストされたパス/ページ。今回の場合は何も指定されなかったのでデフォルトの'/'になります。
* google.comに対するリクエスト用のバーチャルホストが設定されていることを確認します。
* また、サーバはgoogle.comがGETリクエストを受け取れることを確認します。
* さらにサーバはクライアントがこのメソッドを使って良いかを(IPや認証を通じて)確認します。

* Apacheのmod_rewriteやIISのURL RewriteのようなRewriteモジュールがサーバにあれば、リクエストと設定を比較します。もし対応する設定があれば、サーバはその設定にしたがってリクエストの書き換えを行います。

* サーバはリクエストに対応するコンテンツを用意します。今回の場合"/"なのでインデックスファイルです(この設定を上書きすることもできますが、これが最も一般的な方法です)。

* サーバはハンドラにしたがってファイルをパースします。もしGoogleがPHP上で動いていればサーバはPHPを利用してインデクスファイルを解釈し、クライアントに送ります。

ブラウザの裏側
----------------------------------

Once the server supplies the resources (HTML, CSS, JS, images, etc.)
to the browser it undergoes the below process:

サーバがHTMLやCSS、JS、画像などのリソースをブラウザに送ると、以下のようなことがおきています。

* HTML, CSS, JSをパース
* レンダリング - DOMツリーを構築 → ツリーをレンダー → レンダーツリーをレイアウト → レンターツリーを色付け

ブラウザ
-------

requesting it from the server and displaying it in the browser window.
ブラウザの役割は選択したWeb上のリソースをサーバからリクエストし、ブラウザの画面に表示することです。

リソースはHTMLドキュメントのことが多いですが、PDFや画像、またそれ以外かもしれません。
リソースの場所はURI(Uniform Resource Identifier)によって指定されます。

HTMLおよびCSSの既約にしたがってブラウザはHTMLを解釈し表示します。Webの標準化団体であるW3C(World Wide Web Consortium)により、これらの既約はメンテナンスされておいます。


Browser user interfaces have a lot in common with each other. Among the
common user interface elements are:

各ブラウザのUIには多くの共通点があります。たとえば、

* URIを表示するアドレスバー
* 戻るボタンおよび進むボタン
* ブックマーク
* リロードボタンおよび現在のロードをやめるボタン
* ホームボタン

**高レイヤから見たブラウザの構造**

ブラウザの構成要素は:
* **ユーザインターフェース** ここでいうユーザインターフェスは、アドレスバーや戻る/進むボタン、ブックマークなどの、ブラウザのページ部分以外全てです。
* **ブラウザエンジン** ブラウザエンジンは、UIとレンダリングエンジン間の動きを制御するものです。
* **レンダリングエンジン** レンダリングエンジンはレスポンスの内容を表示します。たとえばレスポンスがHTMLならレンダリングエンジンはHTMLとCSSをパースして、スクリーンに表示します。
* **ネットワーク** ネットワークはHTTPリクエストなどのネットワークコールを、プラットフォーム間で共通のインターフェースを通じて行います。ただし、実装自体はプラットフォームにより異なります。
* **UIバックエンド** UIバックエンドはコンボボックスやウィンドウなどの基本的なウィジェットを表示するのに使います。このバックエンドはプラットフォームに依存しないインタフェースをもちます。裏側では、OSのユーザインタフェースメソッドを使っています。
* **JavaScriptエンジン** JavaScriptのコードをパースして実行します。
* **DataStorage** データストレージは記憶層にあたります。ブラウザはクッキーなどに様々なデータを保存できます。ブラウザは、localStorage, IndexedDB, WebSQL, FileSystemなどの保存方法をサポートしています。

HTMLのパース
------------

まずレンダリングエンジンはネットワークレイヤーからコンテンツを取得します。通常、8kBのチャンク単位で行われます。

HTMLのパーサーの主な役割はHTMLマークアップを木構造(parse tree)にパースすることです。

出力された木("parse tree")は、DOM要素とアトリビュートをノードとする木です。ちなみにDOMはDocument Object Modelの略です。DOMはHTMLドキュメントのオブジェクト形式での表現であり、HTML要素のJSなどの外の世界に対するインターフェースでもあります。根は"Document"オブジェクトであり、スクリプトによる操作を行うまでDOMはマークアップと1対1の関係を持ちます。

**パースアルゴリズム**

HTMLは通常のトップダウン、あるいはボトムアップによるパースではうまくパースできません。

理由は次の通りです:

* HTMLは規則がゆるい
* ブラウザは伝統的に有名な無効なHTMLに対してはエラー耐性がある。
* パースの処理は"reentrant"。たとえば他の言語ではパースの最中に入力コードが変わることはないが、scriptタグに`document.write()`の呼び出しがあったりするとトークンが変化することになる。なので、パースの処理自体により入力が変化する。

上のような理由で通常のパース技術が使えないため、ブラウザはHTMLをパースするのに独自のパーサーを利用します。そのアルゴリズムはHTML5既約に詳細に記述されています。

アルゴリズムは大きく2つの段階からなります。トークン化と木構造の構築です。

**パース終了時のアクション**

ブラウザはリンクされた外部のリソース(CSS、画像、JSファイルなど)のフェッチを行います。

この段階でブラウザはドキュメントを操作可能なものとし、遅延評価モードのスクリプトのパースを開始します。遅延評価モードのスクリプトはドキュメントのパース後に実行されます。それが終わるとドキュメントの状態は"完了"状態になり、"ロード"イベントが発火します。

注意すべきなのはHTMLにおいて無効な文法はないというものです。ブラウザは内容の誤りを修正してパースを継続します。

CSSの解釈
------------------

* ”CSS lexical and syntax grammar”にもとづいてCSSファイル、styleタグの中身、styleアトリビュートをパースします。
* 各CSSファイルは"Stylesheet Object"にパースされます。スタイルシートオブジェクトとはセレクタやDOMオブジェクトと、対応するCSSルールをもったものです。
* CSSパーサーは様々ありますが、方式はトップダウンやボトムダウンで構いません。

ページのレンダリング
--------------

* DOMノードをたどって'Frame Tree' または 'Render Tree'を作成し、各ノードのCSSスタイルの値を計算します。
* 'Frame Tree'の各ノードの幅を、子ノードの幅や左右のマージン、ボーダー、パディングを合計してボトムアップで計算します。
* 可能な幅を子ノードに割り当てていくことで、実際の幅をトップダウン式に決めていきます。
* 各ノードの高さをボトムアップで計算します。具体的にはテキストの折り返しや子ノードの高さ、自身のマージン、ボーダー、パディングを考慮に入れて合計します。
* 各ノードの座標を上までの計算結果から算出します。

* 要素が"フロート"だったり、positionが"absolute"や"relative"だったりすると、更に複雑な計算が行われます(http://dev.w3.org/csswg/css2/ や http://www.w3.org/Style/CSS/current-work を見てください)。

* ページのどの部分が"re-rasterized"せずにまとめてアニメーションできるかを示すレイヤーを作ります。各フレーム/レンダーオブジェクトはいずれかのレイヤーに割り当てられます。

* ページの各レイヤにはテクスチャが割り当てられます。
* 各レイヤのフレーム/レンダーオブジェクトはチェックされ、描画コマンドが対応するレイヤに対して実行されます。これはCPUによってラスタライズされるか、GPU(D2D/SkiaGL)によって直接描画されます。

* All of the above steps may reuse calculated values from the last time the
  webpage was rendered, so that incremental changes require less work.
* 上の全てのステップは最後に同じページがレンダーされて際に計算した値を再利用して、少しずつ変化するような変化の計算が簡単になるようにしています。

* The page layers are sent to the compositing process where they are combined
  with layers for other visible content like the browser chrome, iframes
  and addon panels.
* ページのレイヤーは他のiframeやアドオンパネルなどのコンテンツと競合しないように計算されます。

* Final layer positions are computed and the composite commands are issued
  via Direct3D/OpenGL. The GPU command buffer(s) are flushed to the GPU for
  asynchronous rendering and the frame is sent to the window server.
* 最終的なレイヤーの位置が計算され、Direct3D/OpenGLによって複合コマンドが発行されます。GPUコマンドのバッファは非同期的なレンダリングをするためにGPUが担い、フレームはウィンドウサーバーに送られます。

GPU レンダリング
-------------

* 画像計算レイヤはレンダリングの際の計算に、汎用的プロセッサである"CPU"や画像専用プロセッサであるGPUを利用します。

* GPUを画像レンダリング計算に使う場合、画像のソフトウェアレイヤはタスクを小さく分割します。これによりGPUの強力な並列処理能力をレンダリングに必要な浮動小数点計算に対して有効に使えます。

ウィンドウサーバー
-------------

レンダリング後の処理および、ユーザの操作起因の処理
-----------------------------------------

レンダリングが終了すると、ブラウザはJavaScriptを(Google Doodleアニメーションのように)時間差で実行したり、(検索ボックスに文字を入れると候補が出るように)ユーザの操作によって実行します。
FlashやJavaなどのプラグインも実行されるかもしれませんが、Googleのホーム画面の場合はなにもおきません。スクリプトによりネットワークリクエストが送られたり、ページの一部やレイアウトが変化して新たなページレンダリングや描画が行われるかもしれません。

.. _`Creative Commons Zero`: https://creativecommons.org/publicdomain/zero/1.0/
.. _`"CSS lexical and syntax grammar"`: http://www.w3.org/TR/CSS2/grammar.html
.. _`Punycode`: https://en.wikipedia.org/wiki/Punycode
.. _`Ethernet`: http://en.wikipedia.org/wiki/IEEE_802.3
.. _`WiFi`: https://en.wikipedia.org/wiki/IEEE_802.11
.. _`Cellular data network`: https://en.wikipedia.org/wiki/Cellular_data_communication_protocol
.. _`analog-to-digital converter`: https://en.wikipedia.org/wiki/Analog-to-digital_converter
.. _`network node`: https://en.wikipedia.org/wiki/Computer_network#Network_nodes
.. _`varies by OS` : https://en.wikipedia.org/wiki/Hosts_%28file%29#Location_in_the_file_system
.. _`简体中文`: https://github.com/skyline75489/what-happens-when-zh_CN
.. _`한국어`: https://github.com/SantonyChoi/what-happens-when-KR
.. _`downgrade attack`: http://en.wikipedia.org/wiki/SSL_stripping
.. _`OSI Model`: https://en.wikipedia.org/wiki/OSI_model
