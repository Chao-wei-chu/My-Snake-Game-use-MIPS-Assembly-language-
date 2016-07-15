# My-Snake-Game-use-MIPS-Assembly-language
這是我使用MIPS組合語言撰寫的貪食蛇遊戲:)

###貪食蛇說明

		利用”手動貪食蛇”基礎觀念，寫出一個能在一分鐘內自動吃100顆以上食物的貪食蛇AI程式，此過程需:研究->構想->設計->實作。

重點:  
1.	學會研究已完成的CODE，並從中學習。  
2.	學習在實作階段之前，一定要先研究、構思、設計等設計階段應有的步驟，而不是直接實作。  
3.	熟悉MARS Simulator的bitmap和keyboard的使用。  
4.	透過實際操作使我們更了解MIPS(組語)的變化與記憶體、顏色設定的運作模式。  
5.	實作->測試->檢證->修正錯誤->改善or加入新創意->實作。  
6.	了解在有限的時間內，精簡程式、提升效率的重要性(60秒內)。  


###貪食蛇流程圖(Part1:加速&一般模式)  



###貪食蛇流程圖(Part2:Security模式)  
![Security流程圖](/snake_report_security.jpg)

#貪食蛇演算法講解
![圖1-1](/1-1.jpg)

##Part1(加速&一般模式):
###初始設定:
    其實一開始和一般的CODE一樣，先初始化邊框、畫面、畫邊框、畫蛇、隨機產生食物，指示在此我們多加入了初始化旗標，供之後的一般模式和安全模式使用。
###移動前設計:
1.	之後先取得一開始蛇頭的位置和食物位置，之後將以這兩個座標來當蛇移動的依據(若頭的位置與食物相同及判定吃到)，並且在死掉or吃到食物之前不會再更新這兩個座標。

2.	在之後的每次移動裡，要做到幾件事: 
1.取得每次蛇所在位置的尾巴座標 。
2.判斷蛇長是否超過30 格 。
3.取得當下蛇頭的位置(此與一開始取頭的位置是不一樣的) 。 
4.和一般CODE一樣要判定是否死亡、是否growup等。

3.	開始讓蛇往食物座標移動，首先是先判斷headx與foodx的位置，若headx > foodx，則代表食物在蛇的左邊，反之則在右邊；之後再判斷heady和heady的位置，若heady>foody則代表食物在蛇的上面，反之則在下面；之後一前面所判定的位置決定蛇要往上下左右哪邊走(先走y軸再走x軸)。

###移動時設計:
1.	先往判定的方向移動並在移動前確認是否會吃到自己、吃到牆壁，若有則進入”虛擬頭判定”和”追尾巴判定”。

2.	先執行”追尾巴判定”，在”移動前設計”中的第2點中有說，取得目前頭的座標和尾巴座標，若下一步的移動會吃到身體(假設原本要往上)，則改成不要去吃食物，而是去吃尾巴，也就是頭會改往尾巴所在的位置走，以避開吃到自己的風險(利用尾巴座標)，而下一步判定會切回追食物判定。
![圖1-2](/1-2.jpg)
3.	若遇到剛剛好目前頭的座標和尾巴在同一座標上，則會進入”虛擬頭判定”，虛擬頭判定顧名思義就是製造一個假的頭，讓他先一個方向走，若會吃到身體，則往反方向走，同樣的下一步判定會切回追食物判定。(請參考下圖)
![圖1-3](/1-3.jpg)
4.	死亡迴圈判定: 若頭被自己的身體卡住無法走，則設定為已經死亡，並結束程式。(在此我們把final函式的$v0設成10，也可設回32，當初設成10只是為了方便觀察蛇是怎麼死的)(參考下圖)
![圖1-4](/1-4.jpg)

###移動後設計:
    和一般CODE一樣，判斷是否有growup，是否已經死亡，身體移動判斷，初始化等，然後再依情況回到strategy或init。
    
##Part2(安全模式):
###初始設定:
	基本上此模式因為蛇身長度超過30而被執行的，因此是沿用之前的設定，並沒有更新。

###移動前設計: (請對照 P.12圖1.1)
1.	在進入此模式時，其實不會馬上執行security mode 的內容，而是會先強制讓蛇先走到最上方邊緣(圖1.1的邊1(含紅色安全區))，直到確定蛇頭進入安全區域，才開始正式進入security mode。
![圖1-5](/1-5.jpg)
2.	進入安全模式後第一步是先將蛇繞著安全區走一圈，直到走到座標點x=1,y=1才讓蛇開始去吃食物(此步是為了防止蛇進入死亡回圈而讓蛇身先穩穩地貼著安全區走，而不要讓蛇身殘留在安全區外)。
![圖1-6](/1-6.jpg)
3.	當蛇開始吃食物時，我將畫面分成兩個區域，分別是A區和B區(參考圖1.1)，由於AB區皆使用一樣的判斷方式，所以在此只以A區做演算法說明；一開始蛇一定會在A區的左上方，並且此時的蛇不屬於”不貪心模式”(稍後說明)，當蛇要去吃第一顆食物時，他會以最短路徑去吃，並且不會管食物是在A區還是在B區，在吃下第一顆食物後，立即判斷第二顆食物所在的位置，若第二顆食物和蛇都在A區，而且蛇目前的座標尚未超過食物，則再去吃(圖1.7情況二)，否則設定成”不貪心模式”，此模式是讓蛇不再以食物為目標，而是先回安全區域為目標，並且在”不貪心模式”下的蛇會排斥食物，以A區為例，若食物在蛇的左邊則蛇會往右邊(圖1.1邊2)的安全區衝(圖1.7情況三)，若食物在蛇的右邊，則蛇會直接往下衝而不是往左，因左邊是B區而非A區(圖1.7情況一)。
![圖1-7](/1-7.jpg)
4.	B區使用與A區相同但反向的判定方式，即A區是由上而下判斷，B區則是由下而上判斷。(此安全模式複雜的地方就是此處，如何把上面的判斷方式同時放入A區和B區)

###移動時設計:
		基本上和part1:一般模式的判斷方式是一樣的，不過特別拿掉了追尾巴，和虛擬頭模式，而加上了”固定躲避模式”，簡單來說就是蛇會往特定方向躲，之所以拿掉追尾巴和虛擬頭，是因為此安全模式的設計不適用，會走而破壞了原本的安全設計方式。

###移動後設計:
    和一般CODE一樣，判斷是否有growup，是否已經死亡，身體移動判斷，初始化等，然後再依情況回到strategy或init。


