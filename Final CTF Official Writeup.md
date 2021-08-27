# Final CTF Official Writeup

:::warning
**簡報大集合**
[講師們ㄉ血與淚](https://drive.google.com/drive/folders/1QgBklmOl5E7F-PSoSUfVpeYYCCbtyaIc)
:::

:::info
**來搭電梯囉**
[TOC]
:::

## Crypto

### 繞過

:::info
- Description :
	- 小明發現了一個奇怪的加密演算法 (001.cpp)
	一組密文為： vvDtf\[ATbSl\]cFy\[grQloWSYatwEIk\[eYexigte
	你能夠反推演算法的邏輯找到明文嗎？
:::

:::danger
- Hint :
	- 一定要反推演算法邏輯嗎？ 有沒有什麼地方可以“繞過”呢？
	- L字藏?
:::

- 題解
	- 第一題是鄉民福利題
	知道繞過梗的應該是秒解
	- 在講解題方式前先聊聊我出題的心路歷程好了
	這一題我計劃要出一題移項式加密的藏頭
	但藏頭如果出的太明顯就沒意思了
	所以就有了把藏頭寫進程式裡的想法
	既然藏頭一定要用到字串，那就寫一個利用字串進行加密的程式
	在寫的當下我沒有很注意解密是否方便（因為本來就是一題移項式加密的題目），我憑感覺留著一線生機（如果想用邏輯回推還是可行）
	- 接下來講下兩種解題方式
	- 解法1（正解）：
		- Hint有提示去“繞過”程式
	觀察字串可以看到藏頭 PLAINTEXTIS
	FLAG為 Yoyodiy_can_bypass_the_WinRARs_password
	![](https://i.imgur.com/4420NCX.png)
		- 說實話我覺得還蠻明顯的，但卻只有兩個人在沒有開點數hint的情況下解出來
		> （我在main裡面遮掉的string plaintext，X長度我還故意弄得跟yoyodiy那個字串一樣長，幫到這份上，仁至義盡了QQ）
		> [name=Andy]
		- 我覺得可能一來總體題目相當多，大家時間不足，二來這題太少人解，大家下意識地放棄了
	- 解法2：
		- 回推邏輯
		- 觀察程式會發現
		- 這個程式的加密方式為：
			- 明文中的每個字母都會被一個“加密字串”中的某個字母加密
			- 而加密字串是由12個子字串拼接而成（12位鄉民）
			- switch_sequence( )這個函式會改變12位鄉民的順序，導致每次拼接的“加密字串”可能不同
			- 至於明文的字母到底是被“加密字串”中第幾個字母加密，以及12位鄉民如何改變順序
			- 則由tmp這個變數控制，tmp一開始是0，每次都會累積該次的密文字元，越來越大
			- 然而密文是已知的（tmp每次多大也知道了）
			- 所以我們其實知道12位鄉民的順序如何改變
			- 也就知道每次的“加密字串”長什麼樣，以及是“加密字串”的第幾個字母加密明文的
			- 所以現在萬事俱備只欠東風
			- 只要需要利用密文（已知）以及每次的加密字元（已知），就可以解密出明文
			- 但問題是，明文在加密的過程中，使用了“%”（求餘）
			- 這導致不同明文會被加密成同一種密文
			- 說白了就是明文有多種可能
			- 每種可能的明文差距58
			- 還記得密碼盤的同餘嗎？密碼盤是對26求餘，所以每26一循環
			- 這個程式是對58求餘，所以是每58一循環
			- 以下是我之前寫的解密code (001.cpp)
```cpp=
#include<bits/stdc++.h>

using namespace std;


string PTT_popular_netizen[12] = {
    "Patiencel_likes_to_bluff",
    "Leesmart_has_a_good_figure",
    "Asiagodtone_doesnt_like_his_fans_to_laugh",
    "ILoveElsa_likes_to_write_shitpost",
    "Nomorepipe_alert_on_the_epidemic",
    "Typeking_loves_to_make_cheesy_jokes",
    "Eternityring_is_good_at_roadside_banquet",
    "XXXXGAY_four_X_is_equal_to_8000_NTD",
    "Tomchen60229_burns_his_opponent",
    "Ipobar_who_mustn_ot_be_named",
    "Slow_is_the_mother_of_cockroaches",
    "Yoyodiy_can_bypass_the_WinRARs_password"
};

string encrypt_string;

void make_encrypt_string(){
    encrypt_string.clear();
    for (int i = 0; i < 12; i++){
        encrypt_string += PTT_popular_netizen[i];
    }    
}

void switch_sequence(int n){
    if(n == 0 || n == 1 || n == 2){
        for (int i = 0; i < 12; i += 2){
            swap(PTT_popular_netizen[i],PTT_popular_netizen[i+1]);
        }
        
    }else if(n == 3 || n == 4 || n == 5){
        for (int i = 0; i < 6; i++){
            swap(PTT_popular_netizen[i],PTT_popular_netizen[11-i]);
        } 
    }else if(n == 6){
        swap(PTT_popular_netizen[3],PTT_popular_netizen[1]);
        swap(PTT_popular_netizen[4],PTT_popular_netizen[1]);
        swap(PTT_popular_netizen[5],PTT_popular_netizen[9]);
    }else if(n == 7){
        swap(PTT_popular_netizen[2],PTT_popular_netizen[6]);
        swap(PTT_popular_netizen[5],PTT_popular_netizen[3]);
        swap(PTT_popular_netizen[5],PTT_popular_netizen[8]);
    }else if(n == 8){
        swap(PTT_popular_netizen[9],PTT_popular_netizen[7]);
        swap(PTT_popular_netizen[9],PTT_popular_netizen[3]);
        swap(PTT_popular_netizen[2],PTT_popular_netizen[3]);
    }else if(n == 9){
        swap(PTT_popular_netizen[10],PTT_popular_netizen[1]);
        swap(PTT_popular_netizen[11],PTT_popular_netizen[5]);
        swap(PTT_popular_netizen[2],PTT_popular_netizen[3]);
    }
}

int main(){


    string ciphertext = "vvDtf[ATbSl]cFy[grQloWSYatwEIk[eYexigte";

    int tmp = 0;
    char encrypt_letter;
    int plain_letter;
    for (int i = 0; i < ciphertext.size(); i++){
        make_encrypt_string();

        encrypt_letter = encrypt_string[tmp % encrypt_string.size()];

        for (int j = 0; j < 4; j++){
        //for (int j = 1; j < 5; j++){
            plain_letter  = j * 58 + ciphertext[i] - 65 - (encrypt_letter % 58);
            //plain_letter  = j * 58 + ciphertext[i] - 65 - encrypt_letter;
            

            if(plain_letter <= 128 && plain_letter >= 0){
                cout<<char(plain_letter)<<" ";
            }
        }
        cout<<endl;
        
        tmp += ciphertext[i];

        switch_sequence(tmp % 10);

    }
    return 0;
}
```
- 題解 續
	- 執行後得到以下結果
	![](https://i.imgur.com/V7hDRuQ.png)
		- 明文雖然有多種可能
			但稍一觀察還是能馬上看出正確答案
	- 我的解密程式每個明文字元跑4次（每次加58），30個字元共120次
		我不確定這有沒有到最簡以及最正確（僅僅是能用的狀態）
- 講師的題外話
	- 我看了下鯊鯊和Koios1143的code，感覺還是有些太暴力了
	- 而且題目也沒說明文限制在65到122之間，萬一明文裡有數字、符號或不可視字元，就gg了

:::spoiler flag
:::success
**NISRA{Yoyodiy_can_bypass_the_WinRARs_password}**
:::


### 別問，就是幹

:::info
- Description :
	- 題目說明？凱撒加密還需要題目說明？ 別問，就是幹！
	  密文： CNMSZRJITRSAKZRS
 	  請給出明文
:::

:::danger
- Hint :
	- 善用線上爆破工具 或者使用我提供的爆破程式
:::

- 題解
	- 題目說是凱撒密碼，凱撒密碼其中一個簡單的破解方式就是爆破
	- 嘗試26種可能，觀察有沒有可以閱讀的語句
	  > （線上都有提供工具，善用google）
	  > 我看大部分人都有解出來，算是簽到題
	  > [name=Andy]
	  > 
	  ![](https://i.imgur.com/BHKCkhh.png)

:::spoiler flag
:::success
**FLAG:NISRA{DONTASKJUSTBLAST}**
:::

### 基礎波雷費

:::info
- Description :
	- 一題基礎波雷費送給大家
	  密文： SQLEMWNWQFEQNWASICBUQEXGSCUSBTDFXS
	  關鍵字： BASIC
	  請給出明文(全大寫英文)
:::

:::danger
- Hint :
	- 波雷費密碼的加密規則是： 異排異列畫四邊形 同一橫排找右邊 同一直列找下邊
	  那解密呢？
:::

- 題解
	- 題如其名，就是一道很基礎的波雷費密碼
	- 波雷費密碼加解密規則可以參考簡報
		- [簡報連結底加](https://drive.google.com/drive/folders/1QgBklmOl5E7F-PSoSUfVpeYYCCbtyaIc)
		> 懶人包 : 波雷費密碼在第 66 頁開始
	- 需注意的是波雷費密碼在加密的時候，兩個字母同一橫排撞到要找右邊，同一直列撞到要找下邊
	- 解密時則是相反，同一橫排撞到要找左邊，同一直列撞到要上邊
	- 解密完成記得把多餘的X刪掉
	  > 這題使用線上工具可以秒解，但還是希望大家確實瞭解波雷費密碼的規則
	  > （使用線上工具是需注意一些小細節是否相同，例如是否是填充X，5乘5的字母表是否刪除J。線上工具的細節不同，可能會導致不同的解密結果）
	  > [name=Andy]
- 本題題外話
	- 附上我出題時的文本
	> （我出題時都是肉眼一個一個對，不敢完全相信線上工具，以免出錯題目）
	> [name=Andy]
	> 
	![](https://i.imgur.com/sjdNBTd.png)

:::spoiler flag
:::success
**FLAG:NISRA{AREALLYREALLYBASICPLAYFAIRCIPHER}**
:::

### 我吃麥當勞都點兩人份

:::info
- Description :
	- 好想吃麥當勞啊！ 今天就好好吃一頓！這樣明天才有力氣減肥！
	開開心心來到麥當勞，結果發現門口好像在做一個活動。 原本的大門被換成了兩層柵欄。
	店員解釋道，政府即日起開征肥胖稅。肥胖不僅影響身體健康，還會使人思緒遲緩。 麥當勞為了節約稅金，現在只開放夠苗條或夠聰明的人進入。 進入麥當勞的方式有二： 1 穿過柵欄 2 解出一題密碼學題目
	你想著這雙層柵欄肯定攔不住你！ 你看了看柵欄，有些洩氣，雖然只有3米高，但身為肥宅的你是肯定跳不過去了。 欄杆不多只有6根，你側身嘗試擠過去，但肥滋滋的肚腩倔強的卡住了你。
	你搖了搖頭，看了今天想吃到麥當勞只能用你聰明的頭腦了呢！ 誰說肥宅思緒遲緩！
	你拿起了密碼學的題目：
	密文 ：yuedoondeonetgoait
	請給出明文
:::

:::danger
- Hint :
	- 柵欄密碼(可以參考PPT)
	- 「 兩層 」
	- 兩層 3 乘 6 柵欄加密
:::

- 題解
	- 閱讀完題目故事，應該可以看出是柵欄加密，題目故事也有提示柵欄大小是高3寬6，並且是兩層
	- 所以解密就是做兩次 3 乘 6 的柵欄解密（解密方式參考ppt）
		（或者兩次6乘3的柵欄加密）
		（ A乘B的柵欄解密等於B乘A的柵欄加密）
	- 另外，如 Koios1143 的 wirte up 所說，這題也可以用爆破的（工人智慧，雖然很累）
		- 18 個字元的柵欄加密不外乎就是 :
			- 2 乘 9、9 乘 2、3 乘 6、6 乘 3
		- 標準解題過程應該是做兩次3乘6的柵欄解密
- 本題題外話
	- 但 Koios1143 利用線上工具得到只需要做一次 9 乘 2 解密的解法
		（或者說做一次 2 乘 9 的加密）
		> （在這裡感謝Koios1143 ）
		> [name=Andy]
		- 3 乘 6 柵欄加密正常的循環週期依我測試是 16
		- 如下圖
		  ![](https://i.imgur.com/OV8NTuj.png)
			- 假設初始狀態是 state 0
			- 做完兩次 3 乘 6 的柵欄加密得到 state 2
			- 而 state 2 照理要再做 14 次 3 乘 6 的柵欄加密才會回到 state 0
			- 但這時 state 2 做一次 2 乘 9 的柵欄加密就會直接回到 state 0
			  （一個切西瓜的概念）
			> 具體原因非常慚愧我還沒弄明白，這個問題就留給各位了，抱歉
			> [name=Andy]

:::spoiler flag
:::success
**FLAG:NISRA{youneedtogoonadiet}**
:::

### Is it real flag?

:::warning
- 在這裡先感謝會長Mask提供這題，非常血汗的嘗試了無數密明文對，才找到一組加解密前後都有意義的組合，ORZ
	[name=Andy]
:::

:::info
- Description :
	- here u are! : NISRA{eng_be_enc}
:::

:::danger
- Hint :
	- 在很特殊的情況下，有些明文加密後的密文"剛好"還是有意義
:::

- 題解
	- 進行凱撒密碼的爆破
	  > 這題比較通靈，但其實更符合真實的密碼，在實際情況中，加密者會告訴你他是用哪一種方式加密就見鬼了
	  > 在文本長度足夠的時候可以利用IoC或字頻分析，不足時則只能通靈了
	  > [name=Andy]

:::spoiler flag
:::success
**FLAG: NISRA{rat_or_rap}**
:::

### 缺失的設定

:::warning
- 為了確保大家的標準一致 本次密碼學題目中的Enigma
	統一使用： https://cryptii.com/pipes/enigma-machine
	該網站中的 Enigma I 機型
	其中 ROTOR 1 是左轉子，ROTOR 2 是中轉子，ROTOR 3 是右轉子
	反射器使用 UKW-B
	FOREIGN CHARS 設為 Ignore
	RING 設置全部為 A
	不使用Kenngruppen設定
:::

:::info
- Description :
	- 小王:“我再跟你報一遍Enigma設定喔，你把他抄下來。”
	小明：“不用了、不用了，我全都記在腦子裡了。”
	小王:“你確定？”
	小明：“我確定，我記得清清楚楚。”
	小王:“那好，我等下把密文寄過去，你解解看。”
	小明：“小菜一碟！”
	小明和小王在試著利用線上Enigma密碼機，練習Enigma的加解密
	不久，小王發來了另一段由Enigma加密過的密文
	密文： 	sncpdxnztgvfvcfmuigdjvxvmxffiqozmhpvwlfluhpcdsrwlomcuhwrjlqmjowgcglgkioefmaaczqdodwzgnhoulgtm
	小明對著線上Enigma密碼機開始調起設定
	小明：“小王居然還要我把設定抄下來，真是太小瞧我了！”
	小明：“嗯...讓我想想，接線板的設定是...”
	小明：“AU BY CH DQ EF IO JN KL MR PW”
	小明：“我真是太聰明了！這麼長的設定都記得分毫不差。”
	小明：“接著是轉子設定”
	小明：“左轉子是V號備選轉子，中轉子是I號備選轉子，右轉子是IV號備選轉子”
	小明：“左轉子的起始位置是P”
	小明：“右轉子的起始位置是Y”
	小明：“中轉子的起始位置是...”
	小明：“是D......嗎？”
	小明：“還是G...？”
	小明：“GG!我忘記了！”
	過於自信的小明終究還是忘記了一個設定
	但他又拉不下臉重新問小王
	你能幫他解密這則密文嗎？
	註：本題不需“會話密鑰”
:::
- 題解
	- 這題是一題enigma的題目，其實相當簡單
	- 唯一未知的只有中轉子的初始設定
	- 所以先按照題目調整已知的初始設定
	- 然後嘗試中轉子初始設定的26種可能，觀察解密結果，就得到答案
	  ![](https://i.imgur.com/CQDUygF.png)
	  > Enigma加密等於解密，所以密文放在plaintext還是ciphertext其實都一樣
	  > \-
	  > 這題主要是給各位感受一下爆破恩尼格碼並不輕鬆
	  > 缺少一個設定需要爆破26次
	  > 而正常情況是在全部設定未知的狀態下
	  > 需要爆破158962555217826360000次
	  > [name=Andy]

:::spoiler flag
:::success
**FLAG:NISRA{twentysixtriesisnoeasytaskandturinghastomaketentothetwentiethpowerattemptsinmeretwentyminutes}**
:::

### 充太多

:::info
- Description :
	- 小明正在電腦上解一題柵欄密碼的題目 正氣憤著出題者沒有給柵欄的長寬到底要怎麼解
	小王走過來看了一眼說道：“這填充的也太多，太假了！” 小明問道：“你是說我桌面的妹子還是這道題目？” 小王：“都是！”
	小明看著題目，腦中好像有了些想法
	密文： DOYOMANUNTCDPHYAAADNNDDDIYYNAAGNNTDD
	已知密文是明文經過一次柵欄加密後的結果 請給出明文
:::

:::danger
- Hint :
	- 填充的關鍵字是我的英文名字ANDY
:::

- 題解
	- 這題題目有說明是填充過多的柵欄加密
	- 第一種解題方式就是爆破
	- 但我不希望各位爆破，所以故意選了一個因數比較多的數字36
		- 共有2乘18、18乘2、3乘12、12乘3、4乘9、9乘4、6乘6，7種可能
	- 正解是觀察密文尾部的規律，填充的痕跡最容易出現在那裡
	  ![](https://i.imgur.com/DFNRjbV.png)
		- 可以發現尾部呈現X11、X22、X33、X44、X11、X22、X33，這樣的規律
		- 可以合理推斷是以3乘12的格式進行加密
- 本題題外話
	- 這題因為我打算填充ANDY，所以故意選擇寬12的柵欄（ANDY長度為4，是12的因數），使得填充痕跡更加明顯
	- 填充的單詞長度如果不為柵欄寬度的因數（或者說最小公倍數太大），可能沒那麼明顯，肉眼不一定看得出來
	- 但如果填充太多的話，還是會發現密文的IoC明顯過高
	- 以這題為例，其IoC為2.97，遠高於正常的1.73
	  ![](https://i.imgur.com/tX2zvll.png)

:::spoiler flag
:::success
**FLAG: NISRA{DONTPADDINGTOMUCH}**
:::

### 我是誰？

:::info
- Description :
	- CASE 1: 明文 ： WHOAMI
	`凱撒加密   + ☐8 `
	`密文 ：    ITAMYU`
	CASE 2: 明文 ： WHOAMI
	`凱撒加密   + ◯6 `
	`密文 ：    KVCOAW`
	CASE 3: 明文 ： WHOAMI
	`凱撒加密   + ▲2 `
	`密文 ：    KVCOAW`
	CASE 4: 明文 ： WHOAMI
	`凱撒加密   + ▼9 `
	`密文 ：    ZKRDPL`
	請問空缺內應填入什麼數字？
	FLAG形式為：NISRA{☐◯▲▼}
:::

- 題解
	- 這題考得是凱撒密碼的同餘關係
	- 正常解題後會發現，加密的關鍵字為：
		- M(12)、O(14)、O(14)、D(03)
	- 個位數皆不符合題目要求
	- 所以就要利用同餘關係一直加26，直到個位數字符合就是答案
![](https://i.imgur.com/diK74xM.png)

:::spoiler flag
:::success
**FLAG:NISRA{3692}**
:::

### 粗心的保羅

:::warning
為了確保大家的標準一致 本次密碼學題目中的Enigma
統一使用： https://cryptii.com/pipes/enigma-machine 該網站中的 Enigma I 機型
其中 ROTOR 1 是左轉子，ROTOR 2 是中轉子，ROTOR 3 是右轉子
反射器使用 UKW-B
FOREIGN CHARS 設為 Ignore
RING 設置全部為 A
加解密需使用“日密鑰”與“會話密鑰”(參考簡報273-274頁)
不使用Kenngruppen設定
:::

:::info
- Description :
	- 睜開雙眼
	這已經是你穿越回1939年的第10天了
	你莫名其妙的穿越到了一名德軍通訊兵身上，更莫名其妙的是，這個世界的德軍都用英語溝通
	好在你穿越前學過一些密碼學的知識，你勉強能夠勝任工作 你目前在前線負責通訊工作
	今天是12月24日
	德軍計劃今早向對方的陣地發起進攻，失眠的你提早醒了過來
	你拍了拍臉頰使自己保持清醒，等下就要和戰友保羅換班了
	你見到保羅
	還沒來得及打招呼
	只見他一股腦地衝向你，經過你身邊的時候丟給了你一張紙條
	一步不停，邊跑邊大聲叫道：“這是我剛加密完的長官指令，幫我發給後方的炮兵營地！”
	說完一溜煙就不見了
	你看著手中的紙條，笑著搖了搖頭
	這個保羅，每次開打前都肚子疼
	你來到電報機前，正要發送紙條上的密文
	眼角餘光卻瞥見地上掉了一條Enigma接線板上的電線
	謹慎的你意識到有些不對
	你對照著密碼本，一項一項檢查了旁邊的Enigma密碼機
	發現接線板上果然少接了一條電線，這條掉落的電線本應該連 接'A'和'B'
	這代表你手中的紙條很可能是經過錯誤加密的密文！
	德軍馬上就要發起衝鋒，沒時間在路線複雜的戰壕內尋找保羅了
	現在只能靠自己了！
	紙條： nrdfgdeamfqvwoumvpsbhtlrmlkrwe
	請盡快向炮兵營地發送正確的密文(全小寫英文)
	註：請使用與保羅相同的會話密鑰
	![](https://i.imgur.com/jPvmDbw.jpg)

:::

- 題解
	- 首先我們整理已知的資訊，接線板AB連線曾斷開，我們有密碼本和一段可能是經過錯誤加密的密文
	- 我們需要發送正確的密文（在發送密文前要先得到明文）
	- 本題需使用會話密鑰（參考簡報P273，274）
	- 通過題目的日期1939年12月24號我們可以從密碼本上找到當天對應的初始設定
	![](https://i.imgur.com/OTQKd5T.png)
	- 我們先用這個設定解密試試看
	![](https://i.imgur.com/hc68zIS.png)
	- 發現會話密鑰為BUL（前6個字母）
	接著我們繼續用會話密鑰解密剩下的密文
	- 發現解密出一些看不懂的東西
	![](https://i.imgur.com/pDXINjY.png)
	- 拔掉AB連線試試看，發現還是看不懂
	![](https://i.imgur.com/2RRS92e.png)
	- 代表這組會話密鑰可能是錯的
		- AB連線掉落的時間可能更早
		- 我們試著在一開始就拔掉AB連線
	- 得到另一個會話密鑰AUL
	![](https://i.imgur.com/AyJlmup.png)
	- 試著以AUL為會話密鑰繼續解密
	![](https://i.imgur.com/sYPKpaJ.png)
	- 成功了，明文為 attack five minutes earlier
	- 接下來我們只要將 attack five minutes earlier 以正確的方式加密就完成了
	- 第一步加密會話密鑰AUL（記得把AB連線插回去）
		![](https://i.imgur.com/JvdjLXq.png)
		- 得到 前6個密文irdzgd
	- 第二步利用會話密鑰AUL加密明文
		![](https://i.imgur.com/egF6675.png)
		- 得到 cbmmqvwoumvpsahtlrulkrwe
	- 加起來得到正確的密文：
		- irdzgdcbmmqvwoumvpsahtlrulkrwe

:::spoiler flag
:::success
**NISRA{irdzgdcbmmqvwoumvpsahtlrulkrwe}**
:::

## Reverse

**簡易使用 Ghidra 步驟**
1. 需要先建立 project
    - *File* > *New Project* > *Next* > 選好路徑及訂定名稱
2. 將檔案拖進主視窗
3. 出現此視窗後直接按 *OK*
   ![](https://i.imgur.com/1xwzXwB.png)
4. 等他跑
   ![](https://i.imgur.com/imGU47r.png)
5. 出現 *Import Results Summary*，直接打叉或是 *OK* 即可
6. 點兩下剛剛拖曳進去的檔案，或是先選取剛剛的檔案後按上方 *Tools Chest* 的綠龍 ![](https://i.imgur.com/LDxzYCB.png)
7. 出現此視窗後按 Yes
   ![](https://i.imgur.com/fbbhDdC.png)
8. 出現此視窗按 *Analyze*，並等待 *Code Browser* 中左下角沙漏跑完消失
   ![](https://i.imgur.com/NKikM1q.png)
9. 開始使用 Ghidra

:::warning
以下提供的 Code 截圖會視狀況更改變數名稱喔
可利用 點一下要修改的變數 > 右鍵 > *Rename Variable* 達成
:::

### Secret

:::info
- Description :
	- Do you know my top secret?
:::

- 題解
	- 丟到 Ghidra
	- 利用左邊的 *filter* 輸入 `main` 查看
		![](https://i.imgur.com/48hONIL.png =250x250)
	- 在右邊的 *Decompiler* 視窗中看到 main
		- `line 37` 利用 `scanf` 讀使用者的輸入，並將參數放到 `local_78` 中
		- `line 38` `iVar1` 去接 `check(&local_78)` 丟回來的值
		- `line 39 - 44` 若 `iVar1` 為 `1` 即告知 `You got it!!`
		> 上述以白話文講就是將我們的使用者輸入去 check ，若 check 回傳 1，即代表我們的輸入就是 secret 也就是 flag
		> 
		![](https://i.imgur.com/GKtuMNI.png)
	- 深入看 `check` 在做啥 (記住先前他丟了 `local_78` 給 `check` ，代表著他把使用者輸入給了 `check`)
		- 最後會回傳 `uVar2` 回去，且只有那一大坨 `if` 成立時 `uVar2` 才會為 `1`
			- 由此可知那一大坨 `if` 是我們的目標
		- `line 8` 把 `input` 的字串長度放到 `sVar1`
		- 並且由 `line 9` 的 `sVar1 == 0x1e` 得知字串長度為 `0x1e` = `30`
		- 其實他是以不規則的方式去做字串比對，所以只要把每一格是甚麼一一對上去即可，不用寫程式，有紙和筆也可以解，且也不會比寫程式慢
		![](https://i.imgur.com/Q3o5HwN.png)

:::spoiler flag
![](https://i.imgur.com/WGJl5m0.png)
:::success
**NISRA{Y0u_KnoW_t3e_s3cr1t_n0w}**
:::

### Number

:::info
- Description :
	- VGhpcyBpcyBqdXN0IGEgZmFrZSBkZXNjcmlwdGlvbi4gSXQgZG9lc24ndCBtYXR0ZXIgd2l0aCBsYWIuIGxvbGxsbGxsbGxsbGxsbGxsbApKdXN0IHJ1biB0aGUgZWxmLCBzdG9wIGJhc2U2NCBkZWNvZGU=
:::

:::warning
- 看到 `=` 會想到 base64，於是去 decode 之後，會發現根本跟題目沒關係 ~~只是講者不知道要出啥題目~~
  [name=dogQ]
:::

- 題解
	- 丟到 Ghidra
	- 利用 filter 搜尋 main
	- `line 11` 利用 `scanf`  讀使用者的輸入，並將參數放到 `local_14` 中
	- `line 12` 檢查使用者輸入
	   ![](https://i.imgur.com/z4fHQjc.png)
	- 深入看 `check`
		- 整體長得不算太醜，可以看出是做很多次 md5，但可以在 `line 45` 看到關鍵字，而不用去推敲他上面做了甚麼
		- 並注意到 `line 26` 的 `iVar2` 來自 `rand()`，也就是隨機的數字，而 `param1` 是我們的使用者輸入，而我們輸入的數字要等於一個隨機產生的數字是近乎不可能的，所以嘗試直接 patch 此處
		  ![](https://i.imgur.com/coM6nDH.png)
		- 點選要修改的 `if`，而左方的組語會跳到相應的位置
		  ![](https://i.imgur.com/zBpQ0Vi.png)
		  ![](https://i.imgur.com/uPpmHNi.png)
		- 利用 點擊要修改的祖語 > 右鍵 > Patch Instruction 即可修改
			![](https://i.imgur.com/ZsBTyjX.png)
			- 並將邏輯修改相反 (`JNZ` -> `JZ`)
		- 修改完後左上角 *File* > *Export Program*， Format 選擇 `ELF` 並確認路徑及檔名
		  ![](https://i.imgur.com/p268AAc.png)
		- 之後重新運行修改後的程式並隨便輸即可獲得 flag
		  ![](https://i.imgur.com/w65b9Yq.png)

:::spoiler flag
![](https://i.imgur.com/wFLnrQs.png)
:::success
**NISRA{1e962c988d8efc5e5a3853d16329c15a}**
:::

### ccc
:::info
- Description :
	- ccc cc
	- Notice: The flag format is FLAG{abcd, 1234_}
	- From https://hackme.inndy.tw/
		- 這是一個超讚的 CTF 練習平台，有各種類型的題目，推大家練習
:::

:::danger
- Hint :
	- Maybe brute force is a good choice
:::

- 題解
	- 丟到 Ghidra
	- 利用 filter 搜尋 main
		- `line 18` 利用 `read` 讀使用者輸入並最長吃 64(0x40) 個字元
		- 如果 verify 之後回傳的值不為 0 即為 Good
			>代表我們的輸入即為 flag
			>
		![](https://i.imgur.com/IbvEeJ3.png)
	- 來深入看 `verify`
		- `line 10` 代表使用者輸入長度若為 42 就接著進入判斷
		- 接著他會將輸入字串依照 [0:3], [0:6], [0:9]... 持續增長的部分字串丟進 `crc32`，並且將回傳的結果與 `*(int *)(hashes + j*4)` 做比較，若不同則直接回傳 0
			> 其中 `hashes + j*4` 為從陣列開頭+位移量，上述整串可視為取 hashes 中陣列的第幾個資料，意即 `hashes[j*4]`
		>
		![](https://i.imgur.com/yRjSgjz.png)
	- ==撰寫 payload==
	- 取出 `hashes`
		- `hashes` 為程式內本身的資料，可以點兩下直接前往資料所在地
			![](https://i.imgur.com/EVmPXQS.png)
			![](https://i.imgur.com/2qoLjmd.png)
		- 點擊 Display Bytes
			![](https://i.imgur.com/mqhKZJ0.png)
		- Shift + 點擊游標處，應會將整段 `hashes` 選起來
			![](https://i.imgur.com/oR3rNjk.png)
		- 剛剛的 Display bytes 視窗也會被選起來，對綠色框框右鍵 copy 並另外儲存，作稍後比對用
			![](https://i.imgur.com/Bug7e8L.png)
		- 在解析 hashes 時需要有一個小知識，這支程式為 32-bit little endian
			> little/big endian 為電腦中如何處存及解讀資料的方式，有興趣可以查查
			![](https://i.imgur.com/tTuL3bu.png)
		- 附上自己解析的醜醜 code
        ```python=
        f = open("hashes", "r")
        hashes = f.readline()
        hashes_num = []
        print(hashes)

        for i in range(0, len(hashes), 8):
            tmp = ''
            for j in range(6, -1, -2):
                tmp += hashes[i+j:i+j+2]

            hashes_num.append(int(tmp, 16))

        f.close()
        ```
- payload (能跑就好 ><，求大老別鞭)
    - `crc32` 通常用於驗證機制，這邊不多加著墨，Python 中有可以使用的現成函式
    - 所以只要把所有可視字元去做排列組合，並依照原程式的邏輯 ([0:3], [0:6], [0:9],...[0:42]) 丟進去 `crc32` 驗證並與剛剛解析好的 `hashes` 去做比較即可
    ```python=
    import string
    from binascii import crc32

    # Analyse hashes
    f = open("hashes", "r")
    hashes = f.readline()
    hashes_num = []

    for i in range(0, len(hashes), 8):
        tmp = ''
        for j in range(6, -1, -2):
            tmp += hashes[i+j:i+j+2]

        hashes_num.append(int(tmp, 16))

    f.close()

    # brute force

    # discard redundant data
    # due to the known flag length 42 divided by 3 equals 14
    hashes_num = hashes_num[0:14]
    possible = string.printable
    flag = ''
    tmp = ''

    for ha in hashes_num:
        for s1 in possible:
            for s2 in possible:
                for s3 in possible:
                    tmp = flag+s1+s2+s3
                    if crc32(bytearray(tmp, "ascii")) == ha:
                        flag = tmp
                        print(flag)
                        break

    print(flag)
    ```
 
:::spoiler flag
:::success
**FLAG{CRC32 is fun, but brute force is not}**
:::
 
## Operating System

### 1. 來簽到吧

:::info
- Description :
	- 帳密依舊是 nisra 跟 nisra 喔
	  執行這個檔案 : /home/nisra/Welcome
	  btw 這次沒有sudo給的root權限了各位加油
:::

:::danger
- Hint :
	- 執行檔案：`./<filename>`
	  更改權限：`chmod <permission> <filename>`
		> 這邊的 \<xxx> 是示意，別真的打嘿
:::

- 本題關鍵
	- `./home/nisra/Welcome`
	  權限 : 755
	  ![](https://i.imgur.com/gG5XNKD.png =400x300)
	  ![](https://i.imgur.com/8L7k8PY.png =400x300)
	  > 幫各位快速複習一下甚麼是755
	  > 圖截自OS上課簡報
	  > 忘記ㄉ快回去複習!
    - 指令
		- 執行檔案 `./<filename>`'
- 題解
	1. `./home/nisra/Welcome`
	   ![](https://i.imgur.com/kC6l6OI.png)

:::spoiler flag
![](https://i.imgur.com/Zya9ddI.png)
:::success
**NISRA{Hell0Wor1d!!!}**
:::	 

### 2. home directory

:::info
- Description :
	- 沒錯這個系統也有個 lock 使用者
	  去 lock 使用者的 home 目錄看看吧~~
:::

:::danger
- Hint :
	- 變更目錄 `cd <path>`
	  檔案 or 目錄：`ls <parameter, optional>`
	  你確定你看到的東西就是你想像的那樣?
	  噓)系統有幫你高光喔www)
:::

- 本題關鍵
	- `/home/lock` 然後他是個檔案
	  權限 : 755
	- 指令：
		- 變更目錄 `cd`
		- 檔案 or 目錄：`ls`
- 題解
	1. `cd /home/lock`：
	   如果還記得上課內容的話應該會先想這樣做
	   但是進不去因為他是檔案
	2. `cd /home`：
	   先退到 `/home` 準備下一步
	   ![](https://i.imgur.com/szpzL4B.png)
	3. `ls -la`：
	   看看 `lock` 到底是什麼
	   發現是檔案先跑起來再說
	   ![](https://i.imgur.com/axgjHqd.png)
	4. `./home/lock`：跑起來~
	   ![](https://i.imgur.com/ZMPySYp.png)

:::spoiler flag
![](https://i.imgur.com/DhzZBCz.png)
:::success
**NISRA{0MG_I_hav3_n0_h0m3}**
:::

### 3. cl4ssifi3d_fi1e

:::info
- Description :
	- `nisra` 的 `home` 目錄有個咚咚等著你挖掘~
	  (喵~
:::

:::danger
- Hint :
	- 看到檔案別總是想著執行檔案
	  想想看貓跟這題會有甚麼關係
	  (等等又有人ㄑㄧㄠ-
	  ...
	  (喵~
:::

- 解題關鍵
	- `/home/nisra`
	  權限 : 755
	- `/home` 這個資料夾
	  有上課應該會有點印象
	  這個資料夾等同 Windows 底下的 *使用者* `Users` 資料夾
	  下載、音樂、影片、桌面都在這
- 題解
	1. **`cd /home/nisra`**：
	   先回家
	2. `ls -l`：
	   會看到一個超明顯的 `c0nfid3nti4l/`
	   ![](https://i.imgur.com/Hasdu6G.png)
	3. `cd c0nfid3nti4l/`：
	   進去看看
	4. `ls -l`：
	   看到目標 `s3nsitiv3_d4ta`
	   ![](https://i.imgur.com/SJZ5meq.png)
	5. `./s3nsitiv3_d4ta`：
	   然後看到只有一半的 Flag
	   ![](https://i.imgur.com/aLxniiC.png)
	6. `cat s3nsitiv3_d4ta`：
	   再用 `cat` 看裡面有什麼東西 ~是`lock`\ 這個使用者的密碼~
	   ![](https://i.imgur.com/oghRTbi.png)

:::spoiler Flag
![](https://i.imgur.com/evp8unt.png)
:::success
**NISRA{NOPElalala_iz_lock's_passw0rd}**
:::

### 4. 使用者們

:::info
- Description :
    - 看到上一題的Flag相信lock的密碼已經出來啦
	  這次的答案藏在nisra的home目錄下的某個角落
	  絕對明顯(廢話不然勒(X
	  (解完的也可以看看hint我有塞一點東西進去喔~)
:::
	
:::danger
- Hint :
	- `ls`後面能接什麼還記得嗎~
	  問小天使之前
	  先試試`ls --help >> temp.txt`
	  (這個代表把ls --help的結果印到temp.txt
	  (ls的說明太長了直接打看不到全部說明
	  然後用vim打開temp.txt看看~(`vim temp.txt`)
:::

- 解題關鍵
	- `/home/nisra/.Hey_u_YES_U_OVER_HERE_dammit/.Remember_how_to_read_me?`
	  權限 : 700
	- 指令
		- 切換使用者：`su`
		- `ls` 的後綴 `-la`
		- 也許 `vim` ? 看有沒有看 hint
- 題解
	1. `ls -la`： 用 `-a` 後綴把隱藏檔撈出來
	   ![](https://i.imgur.com/lbFds2P.png)
	2. `cd .Hey_u_YES_U_OVER_HERE_dammit`： 就說了很「明顯」了
	3. `ls -la`： 發現一個擁有者為 `lock` 的檔案 `.Remember_how_to_read_me?`
	   ![](https://i.imgur.com/a9qC0Iu.png)
	4. `su lock`： 用剛剛的密碼登入 `lock`
	   ![](https://i.imgur.com/R3txTvM.png)
	5. `./.Remember_how_to_read_me?`： 跑起來
	   ![](https://i.imgur.com/5BVOJJb.png)

:::spoiler Flag
![](https://i.imgur.com/JkWMl81.png)
:::success
**NISRA{A1most_d0ne_k3ep_up}**
:::

### 5. Grand Finale

:::info
- Description :
	- 恭喜你撐到最後一題啦 ~
	  這題的Flag散落於前幾題
	  試著拼出原貌吧 ~
:::

:::danger
- Hint :
	- 仔細看前幾題的題目與檔案(wink wink
	  順便提醒雖然`cat`方便但是`vim`有高光功能
	  挑一個喜歡的用吧~
:::

- 解題關鍵
	- 就只是把Flag拆開塞到每一題裡面而已
	  把上面那幾題cat出來拼起來
- 題解
	1. **`cat /home/nisra/Welcome`**
	   PART 1 : `NISRA{C0ngr4ts!`
	   ![](https://i.imgur.com/zu3XtZI.png)
	2. **`cat /home/lock`**
	   PART 2 : `N3xt_`
	   ![](https://i.imgur.com/wKiq9cZ.png)
	3. **`cd /home/nisra/c0nfid3nti4l`**
	   **`cat s3nsitiv3_d4ta`**
	   PART 3 : `S3ction_`
	   ![](https://i.imgur.com/e0Z18JT.png)
	4. **`cd /home/nisra/.Hey_u_YES_U_OVER_HERE_dammit`**
	   **`su lock`**
	   **`cat .Remember_how_to_read_me?`**
	   PART 4 : `Plz}`
	   ![](https://i.imgur.com/Ci1wPNw.png)

:::spoiler Flag
:::success
**NISRA{C0ngr4ts!N3xt_S3ction_Plz}**
:::

### 6. Or is it?

:::warning
別他喵的給我偷偷看Root密碼阿可惡
~~對我就是在說你某鯊w~~
~~我就是要強制讓他們操作Terminal~~
~~不服不打架~~
[name=404]
:::

:::info
- Description :
	- 好啦這題只是想告訴你們 `root` 密碼是 `Enlightened`
	  兩台虛擬機都是
	  加上這台的正常關機要用 `root` 打 `systemctl poweroff`
      (也可以直接斷電啦但不推薦)
	  去拿白嫖10分吧：`./root/10pt_reward~`
	  Tips : `/root` 是 `root` 使用者的home目錄喔不在 `/home` 裡面
:::

:::danger
- Hint :
	- U Serious?
:::

- 題解
	1. `su root`：
	   先登入 `root` 
	   畢竟是 `root` 的 `home` 目錄人家沒義務開權限給你
	   密碼是 `Enlightened` ~~(都寫在題目裡了應該知道啦~~
	   ![](https://i.imgur.com/qg7Tn7Z.png)
	2. `./root/10pt_reward`：
	   再來直接拿題目給的路徑 `./root/10pt_reward~` 去挖~~白嫖~~flag
	   ![](https://i.imgur.com/zqjAvIz.png)
	3. `systemctl poweroff`：
	   既然都提到了要怎麼關機那就去關機啊
	   別給我直接斷電阿可惡會出事阿
	   打 `systemctl poweroff` 會真的關機
	   ![](https://i.imgur.com/qIVi60I.png)
	   > 然後就關機惹

:::spoiler flag
![](https://i.imgur.com/lN6ts9p.png)
:::success
**NISRA{G4ME_0V3R~}**
:::

## Info Leakage

### Beauty

:::warning
- hashtag
	- #通靈王爭霸賽
	- #絕殺
	- #進巨好看嗎XDD ~~@豬排咖哩飯~~
:::

:::info
- Description :
	- 感恩IG有典藏功能，不然采婷常常發文後又後悔
:::

:::danger
- Hint :
	- Instagram密碼
:::

- 題解
	- 因為題目說要在典藏找貼文嘛，但是典藏只有帳號主人可以看
	- 所以我們要來登錄采婷ㄉIG帳號
	- 這題用的是~~我們以為很簡單的~~弱密碼
		- `instagramau4a83`
		- 也就是 instagram 密碼
			- 密碼 -> au4a83
				> 所以說提示要好好ㄎㄢˋ-$%#%&#@**/ˇ%
				> [name=被大家圍起來揍的LAVI]
			-  關於提示寫 **I**nstagram 但 密碼實為 **i**nstagram
				> 拍謝，下次會小心
				> [name=CHA]
			- 成功登入采婷的IG後點右上角三條橫線的地方可以看到典藏貼文的選項
			  ![](https://i.imgur.com/8iwGEOq.png =300x400)
			- 找到這裡的那篇貼文就是我們藏flag的地方ㄌ
			  ![](https://i.imgur.com/mshu15c.png =400x300) 
	> p.s. 為了避免采婷~~被糟蹋侵犯~~受到大家愛護，已經改密碼 + 停用帳號，找不到她 & 登不進去是正常的哦～請好好想她，以後才有機會再出現(？

:::spoiler flag
![](https://i.imgur.com/7EGHRSr.png =300x400)
:::success
**NISRA{5eCr3t_p0sT?}**
:::

## Web

:::warning
- browser 用的是 chrome，edge 等其他的是否能呈現一樣的效果並不確定。
  [name=雩生]
:::

### Basic

:::info
- Description :
	- http://chall2.nisra.net:41001/
:::
![](https://i.imgur.com/J7paLbo.png)
- 題解
	- 根據提示，可以猜測 flag 藏在 html，css，JavaScript裡
	- 按 f12 或是滑鼠點擊右鍵，選擇最下方檢查，即可開啟介面
	- 點開 source 可以發現就簡單的三個檔案，點開往下翻就能找到 flag
	- 這裏由於題目沒有提示這三份拆開的是以什麼順序排序，所以所以就以前後括號來判斷，順序是 html -> css -> JavaScript

:::spoiler flag
- html
  ![](https://i.imgur.com/KTXuzym.png)
- css
  ![](https://i.imgur.com/lj9WB7e.png)
- JavaScript
  ![](https://i.imgur.com/at00uG1.png)
:::success
**NISRA{Th3r3_is_sti11_4_10n9_w4y_t0_th3_w36}**
:::

### Keyboard

:::info
- Description :
	- http://chall2.nisra.net:41002/
:::

- 題解
	- 打開後會看到格子和一個藍點以及一串不明意義文字
	  > （理論上）藍點可以透過上下左右鍵操控
	  > 
	  ![](https://i.imgur.com/7axQhQR.png)
	- 按 f12 看一下，會看到有一段被註解掉的 js，仔細看一下會看到關鍵字 keycode，若是對 js 語法比較了解的應該能看出這是在監聽鍵盤按鍵
	  ![](https://i.imgur.com/hv3lt8c.png)
	- 這時回憶一下那串不明意義符號，找一下符號的 ASCII code
	  ![](https://i.imgur.com/o7fxqaW.png)
	- 會發現這些符號的十進位數字，可以剛好對應到 keycode 裡上下左右鍵的數字
	  ![](https://i.imgur.com/y3X9vEx.png)
	- 分別對應完畢後在畫面上按壓（⬆️⬇️⬅️⬅️⬇️➡️⬆️）即可得到flag

:::spoiler flag
![](https://i.imgur.com/3pm16de.png)
:::success
**NISRA{Y0u_93t_itTTTTTTTTTTT}**
:::

### Length

:::info
- Description :
	- http://chall2.nisra.net:41003/
:::
- 題解
	- 點開後閱讀文字，能得知如果要拿到 flag，要在輸入框輸入一個大於 12 的數字
	![](https://i.imgur.com/xmfgxdr.png)
	- 嘗試輸入後會發現只能輸入一位，開啟原始碼觀察一下，會在 html 表單的 input 內看到關鍵字 maxlength=1
	  ![](https://i.imgur.com/wa7ZWlp.png)
	- 當下不知道 Maxlength 作用的可以直接查找資料，然後就能發現就是這個關鍵字限制了輸入長度
	- 滑鼠移到輸入框的 input 上，右鍵後選擇 edit Html 就可以編輯內容，將 1 修改成其他更大的數字
	  ![](https://i.imgur.com/v6cIzxI.png)
	- 然後點一下其他地方，再在輸入框輸入一個大於 12 的數字，就能拿到 flag

:::spoiler flag
![](https://i.imgur.com/pLk4iPN.png)
:::success
**NISRA{D0_y0u_kn0w_m4x13n9th}**
:::

### Robots
:::info
- Description :
	- http://chall2.nisra.net:41004/
:::
:::danger
- Hint :
	- https://awoo.ai/zh-hant/blog/robotstxt-crawl/
:::

- 題解
	- 如題，對應第四天的課程內容，可以想到說這個藏東西的地方應該就是在 robots.txt 內
	  ![](https://i.imgur.com/s2Zl2KA.png)
	- 在 robots.txt 可以找到另一個 html，進入後就能得到flag
	  ![](https://i.imgur.com/b07vXll.png)

:::spoiler flag
![](https://i.imgur.com/B52CTVP.png)
:::success
**NISRA{Robots_h4s_s0methin9s}**
:::

## Steganography

### Header 0x1

:::warning
- 抱歉ㄌ鯊鯊，看來我們不是 soulmate QQ
  [name=LAVI]
:::

:::info
- Description :
	- How to open the file?
:::

- 題解
	- 起手式 : 丟到 WinHex / 010 editor
	  ![](https://i.imgur.com/sBpfGSZ.png)
		- 從 header 可以發現原來他是 pdf 檔
			- pdf 的 header 是 `25 50 44 46 2D`
			- 也可以從ASCII那裏直接看出
	- 把副檔名改成 `.pdf` 
	  ![](https://i.imgur.com/kjEQ9JQ.png)![](https://i.imgur.com/cRmjp2h.png)
	- 改好檔案後可以雙擊開啟檔案
		- 接著在紙上全選反白flag複製
	  ![](https://i.imgur.com/VSb1qPA.png)

:::spoiler flag
:::success
**NISRA{w1nh3X_1s_be5t_to0l}**
:::

### Header 0x2

:::info
- Description :
	- What kind of file is this?
:::

- 題解
	- 一樣起手式丟到 WinHex / 010 editor 
		- 檔名都告訴你這是一個 png 檔了，那就直接改ㄅ
		   > 判斷檔案格式方式之二 : 
		   > 其實丟到工具後你也會發現他 ASCII 那邊有寫 IHDR
		   > 這是 PNG 才有的
		- 把 header 改成 png 的 `89 50 4E 47 0D 0A 1A 0A`
		   ![](https://i.imgur.com/xQQmYv9.png)
		   > 不要用複製貼上的哦，自己乖乖手打辣(X
		- 再去改副檔名
		   ![](https://i.imgur.com/PPWnqaO.png)
		- 就可以看到flagㄌ!

:::spoiler flag
![](https://i.imgur.com/ZTsaWs8.png)
:::success
**NISRA{w1nh3X_1s_be5t_to0l}**
:::

### Bin x walk

:::info
- Description :
	- pictures
:::

- 題解
	- 根據題目，可以猜到我們這題會用到 Binwalk
	- Binwalk起手操作 : 進到 Binwalk 資料夾後在路徑的地方打 `cmd` 可以直接把 cmd 路徑都開好 ( 新手如果不太熟悉cmd操作推薦使用此方法 )
      ![](https://i.imgur.com/dwNbhpL.png)
      ![](https://i.imgur.com/RfTrxwK.png)
	  > 在這裡打 `cmd`
	- 接著如同上課說的那樣，輸入指令
		- python src\scripts\binwalk -e <File>
		- python src\scripts\binwalk -D=".*" -C="<Folder>" <File>
			- -D=".*" all type
			- -C="<Folder>" output folder
	- 再來在 binwalk 裡打開你新取名的資料夾( 像我這邊取名叫做 `nisra`  ) / 或是直接打開 `_Lab_Binwalk.jpg.extracted` 這個資料夾
	- 前者進到資料夾後，替目標檔案新增副檔名 `.zip` 後進行解壓縮後會產生 flag 
	  ![](https://i.imgur.com/K68N9cR.png)
	- 後者直接打開資料夾就會看到 flag 了
	  > 都是預期解
	  > 能解出 flag 的方法都是好方法

:::spoiler flag
![](https://i.imgur.com/deyPvez.png =450x300)
:::success
**NISRA{pix_loves_zip}**
:::

### Mysterious Words

:::info
- Description :
	- Hidden flag in Docs
:::

- 題解
	- 開啟 word 檔案後可以到左上角 "檔案" 找到 "選項"
	  ![](https://i.imgur.com/OltzUKa.png =300x300)
      ![](https://i.imgur.com/oWr7WKe.png =400x500)
	- 找到 "顯示" 後在 "在螢幕上永遠顯示這些格式化標記" 中勾選 "隱藏文字" 然後按確定
	  ![](https://i.imgur.com/sudGOn3.png)
	- flag 就出現ㄌ	  

:::spoiler flag
![](https://i.imgur.com/Fs3O8FO.png)
:::success
**NISRA{DOC_X_WORD}**
:::

### QR code Master

:::info
- Description :
	- 拼圖！
:::

- 題解
	- 本題所需觀念 
		- QR code有固定格式特性用於辨識定位標記
			-  詳細可見 wiki [QR 碼](https://zh.wikipedia.org/wiki/QR%E7%A2%BC#%E6%8A%80%E6%9C%AF%E7%89%B9%E6%80%A7)
		   ![](https://i.imgur.com/wvCWis4.png =400x300)
	- 大概了解該拼成怎樣後就可直接打開你的好朋友開始拼圖!
		- 這題蠻容易的判斷3個大"回"分別在左上、左下、右上角而且沒有經過任何旋轉誤導切塊之類的~~減少大家通靈的時間~~
		  > 原本有做一個切的超破碎的版本，但後來覺得太殘忍了就沒放
		  > 
			![](https://i.imgur.com/BtowY84.png =300x300)

::: spoiler flag
![](https://i.imgur.com/0I99It6.png)
:::success
**NISRA{9uick_r35p0nse_Code}**
:::

### Header 0x3

:::info
- Description :
	- Where is the header?
:::

- 題解
	- 這題我們直接打開後就可以發現他竟然沒有檔頭
		![](https://i.imgur.com/BbmdRv5.png =500x300)
	- 所以我們要用插入的方式為他新增檔頭
	- 在 Edit 選擇 Insert / Overwrite 裡的 Insert Bytes
		![](https://i.imgur.com/qWMzFiH.png)
	- 再來如下圖
		- 因為我們已知 header 的位址，所以 Start Address 設成0 
		- 而 png 的檔頭 `89 50 4E 47 0D 0A 1A 0A` 佔用了 8 個 btyes，所以在 Size 選 8 
		- 最後因為這些都是16進位制，故選擇 Hex
		![](https://i.imgur.com/O03tqpz.png)
 	- 點下 Insert 後產生的結果如下圖
		![](https://i.imgur.com/PKhQjLu.png)
	- 這時候直接幫他改成 png 的檔頭後~~記得按儲存~~就可以挖到 flag 了
		![](https://i.imgur.com/7nDI4yI.png)

:::spoiler flag
![](https://i.imgur.com/NjoaYQ7.png =450x300)
:::success
**NISRA{wh3re_1s_my_m4g1c_num6errrrr}**
:::

### Secret Document

:::info
- Description :
	- Does the file need a password?
:::

:::danger
- Hint :
	- Pseudo Encryption
:::

- 題解
	- 此題考點：zip偽加密
        > 出完題目後有發現 7z 等解壓縮軟體可以繞過
        > 但想說和拼圖一起當送分題就不改了 QAQ
        > 都佛系出題 + 花式送分了還不給我多解一點
        > [name=CHA]
        - [ZIP File Format Specification - Version 6.3.6](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT)
        - 4.3.6 Overall .ZIP file format
            - 關於此題比較重要的是
                1. `Local File Header`
                2. `Central Directory Header`
        - 4.3.7 Local File Header & 4.3.12 Central Directory Structure
            |                   Name                   | Bytes |             Name              | Bytes |
            |:----------------------------------------:|:-----:|:-----------------------------:|:-----:|
            | Local File Header Signature <br>(0x04034B50) |   4   | Central File Header Signature <br>(**0x02014B50**) |   4   |
            |        Version Needed To Extract         |   2   |        Version Made By        |   2   |
            |       **General Purpose Bit Flag**       | **2** |   Version Needed To Extract   |   2   |
            |            Compression Method            |   2   | **General Purpose Bit Flag**  | **2** |
            |                   ...                    |  ...  |              ...              |  ...  |
        - 4.4.4 General Purpose Bit Flag
            - `Bit 0: If set, indicates that the file is encrypted`
                $\because$ 2 bytes = 16 bits = $2^{15}...\ 2^0$
                $\therefore\ 2^0\rightarrow$ 1 or 0 $\rightarrow$ Odd or Even
            - 此區塊若為**奇數**，檔案便被視為**有加密**
        - 判別條件
            - [7z SRC](https://github.com/mcmilk/7-Zip)
                - 有興趣的可以去挖 Source Code
            - $\because$ 此題只有將 `Central Directory Header` 的 `General Purpose Bit Flag` 設為奇數
                $\therefore$ 不同解壓縮軟體 ${\implies}$ 不同的識別方法 & 解密方法 ${\implies}$ 需不需要輸入密碼
	- 此題解法
		- 首先我們起手式，用 010 editor 打開我們的壓縮檔
		- 接著利用 `ctrl + F` 去找尋 `50 4B 01 02` 所在位址 (即為Central File Header Signature)
		  ![](https://i.imgur.com/yzuv1Ly.png)
		- 並找到其後數來第 5 、6 個 byte `05 00`，將其修改回 `00 00`，如此一來便不需要密碼就可以解壓縮了  
		  ![](https://i.imgur.com/sKf6A8I.png)

:::spoiler flag
![](https://i.imgur.com/EdvHabT.png =450x300)
:::success
**NISRA{Z1p_d0e4nT_h4Ve_PasSwD}**
:::



###### tags: `NISRA` `Enlightened` `2021`