# Chapter 2 有意義的命名
## **讓名稱代表意圖-使之名符其實**
變數、函式或類別的名稱，要能夠告訴我們，它為什麼會在這裡出現，、它要做什麼用、還有該如何使用，如果名稱還需要註解，那它就不具備展現意圖的能力。
```
int d; //消逝的天數

//上面的宣告，並沒有明確的表達變數用途
//改成下面的名稱會更好

int elapsedTimeInDays;
```
上面一行只是變數的簡單命名，那如果變成一段程式碼呢 :
```
public List<int[]> getThem() {
	List<int[]> list1 = new ArrayList<>();
	for (int[] x : theList)
		if (x[0] == 4)
			list1.add(x);
	return list1;
}
```
雖然程式碼的邏輯、縮排、變數數量都不複雜，但問題出在程式碼的意圖並不夠明顯 :
* `theList`放著什麼類型的東西 ?
* `theList`裡索引0的項目是什麼 ?
* 數值`4`的意義是 ?
* 該如何使用回傳的`List` ?

想像一下，如果要開發踩地雷遊戲，那這段程式碼，就可改成下面的樣子 :
```
public List<int[]> getFlaggedCells() {
	List<int[]> flaggedCells = new ArrayList<>();
	for (int[] cell : gameBoard)
		if (cell[STATUS_VALUE] == FLAGGED)
			flaggedCells.add(cell);
	return flaggedCells;
}
```
更進一步，將地雷格int[]用一個類別封裝，再次增加整體的可讀性 :
```
public List<Cell>> getFlaggedCells() {
	List<Cell> flaggedCells = new ArrayList<>();
	for (Cell cell : gameBoard)
		if (cell.isFlagged())
			flaggedCells.add(cell);
	return flaggedCells;
}
```
能看到選一個好名稱，就可以表達不少事情。

## **避免誤導**
而除了挑選一個好名稱，也盡量避免會誤導人的線索，比如 :
* 不要挑選`hp`、`aix`、`sco`等名稱，它們是Unix平台的名稱，即使在撰寫有關三角形斜邊的程式。
* 不要用`accountList`來代表一群帳戶的變數，除非變數型別真的是`List`。
* 不要用難以分辨的名稱當作變數名稱
`XYZControllerForEfficientHandlingOfString` vs. `XYZControllerForEfficientStorageOfString`、
小寫`L` vs. 數字`1`、大寫`O` vs. 數字`0`

## **產生有意義的區別**
有意義的命名之後，就得看在眾多有意義的變數當中，可以區分出它們之間的**區別**，來看一些反向的例子，其中一個，就是數字序列命名法 :
```
public static void copyChars(char a1[], char a2[]) {
	for (int i = 0; i < a1.length; i++) {
		a2[i] = a1[i];
	}
}
```
如果把`a1`、`a2`分別改成`source`、`destination`，明顯的這兩個參數就有所不同，還能展現意圖。

另一個是添加**無意義**的字詞在變數裡面，分成兩種 : 
* 添加了單純沒有用的字詞在變數裡，像是`variable`
* 想像有個`Product`類別，還有個`ProductInfo`、`ProductData`類別，這些類別根本分不出差別在哪，`Data`、`Info`並沒有區分的效果，還有很多類似的例子，`a` vs. `the` ，或是下面的方法 :
	```
	getActiveAccount()
	getActiveAccounts()
	getActiveAccountInfo()
	```
## **使用能唸出來的名稱**
在定義上，字詞應該要可以發音，如果不能夠被發音，對於我們在閱讀、討論上其實多少都會有障礙，導致編寫程式碼的時間會更久。

## **使用可被搜尋的名字 ( 單一字母 & 數字常數 )**
比起搜尋數字7，搜尋`MAX_CLASSES_PER_STUDENT`會更加容易，另外變數名稱若只是單一字母 ( 好比英文字母`e` )，也會增加搜尋難度，`e`幾乎出現在每個地方。因此把握一個原則 : **命名的長度應與其scope大小相對應**。

## **避免編碼**
這邊的編碼，指的是在變數的前面，加上一些前贅字來代表它的某種特性，好比在成員變數前加上`m_`來區分不同，但現在的IDE是能夠將這種變數**變色**凸顯出來，因此這樣的前贅字似乎就變得不太需要。

而`Interface`和`Implementations`，過去在命名上有在前面加上`I`和`C`的例子，看起來會像是`IShapeFactory`和`CShapeFactory`，很多時候看到這種前贅字，我們會自動忽略只看後面重要的部分，因此與其用這樣的命名，不如使用`ShapeFactory`和`ShapeFactoryImpl`吧。

## **避免思維的轉換**
命名應該要精準，要使用問題領域的命名，不該出現閱讀程式碼的人還要在腦中把名詞轉換過的情況。

當然像是迴圈記數，會用慣例用法只用單一字母 ( 不過永遠不要使用l )，但其他的情況，命名越清楚越好。

## **類別的命名**
類別跟物件應該使用名詞或名詞片語，不該使用到動詞。

## **方法的命名**
方法應使用動詞或動詞片語來命名，而根據javabean的標準，取出器 ( accessors ) 應使用`get`當字首、修改器 ( mutator ) 應使用`set`當字首、判定 ( predicates ) 應使用`is`當字首。

另外，當建構子被多載 ( overloaded ) 時，可以使用含有參數名稱的靜態工廠方法，順便把建構子設定成`private`，凸顯封裝性 :
```
Complex fulcrumPount = new Complex(23.0); 
Complex fulcrumPount = Complex.FromRealNumber(23.0); //better
```

## **不要裝可愛**
不要用俚語、俗語、或是其他賣弄小聰明的名稱來命名 :
* `HolyHandGrenade` vs. `DeleteItems`
* `whack()` vs. `kill()`
* `eatMyShorts()` vs. `abort()`

## **每個概念使用一種字詞、別說雙關語**
替單一抽象概念挑選一個字詞，並堅持持續地使用它。比如在命名**不同類別的取得方法時**，採用`fetch`、`retrieve`、`get`這些不同名稱，就會讓人感到困惑。

而雙關語就是字面上的意思，比如原有的類別有個`add`方法，是用來相加兩個現有的值，後來又新增了一個類別，裡面有個方法是把參數放到集合中，那使用`add`來命名，就不是一個很好的選擇，可以改用`insert`或`append`會更好。

## **使用解決領域方案的命名、使用問題領域的命名**
若閱讀程式碼的人，同樣是程式設計師，那就使用電腦科學領域的用語，使用問題領域的名稱，那協同工作者就必須來回跟客戶確認，每個字詞的含意。不過如果找不到程式設計師熟悉的用語，那退而求其次就是使用問題領域的命名，至少維護時還能詢問該領域的專家，名稱所代表的意思。

## **添加有意義的上下文資訊**
只有少部分的命名，能由命名本身了解足夠意義，大部分都必須要有上下文資訊，好比當看到`firstName`、`lastName`、`street`、`houseNumber`、`city`、`state`、`zipcode`，我們可以認出這是地址，但如果只看到`state`，有辦法直接認出來嗎 ?

當然可以利用字首來增加資訊，像是`addrFirstName`、`addrLastName`、`addrState`等，不過更好的做法是，建一個Address類別。

## **別添加無理由的上下文資訊**
想像一下，如果有一個應用程式叫**豪華版加油站 ( Gas Station Delux )**，在這個應用程式裡面，把每個類別都加上`GSD`字首，如果我們按下`G`，再讓IDE來幫我們搜尋關鍵字類別，那會把每個類別都找出來，這樣就是沒有意義的上下文資訊，裡面有三個字母沒有任何意義。

## 尾加碎碎念
在第一次閱讀這章時，他本身所想強調的核心概念**有意義的命名**，這件事情其實本來就有所耳聞，因此大部分所舉出來的例子，也不會讓我太過訝異。不過章節一開始所展示的程式碼，感覺上跟核心概念比較沒關係，但倒是讓我對於物件的封裝，有些特別的想法。

它是用踩地雷的程式碼當作的例子，把`int[]`這樣型態的物件，特別使用一個`Cell`的類別封裝起來，雖然可能對很多人來說，這樣的封裝可能跟一般老掉牙的`Point`例子完全一樣，但對我來說有些啟發 : 即使單純的使用`int[]`這樣的物件，就可以完全達到目的，而且也程式碼看起來也不會太過複雜，然而多做一層`Cell`的包裝，卻對整體的可讀性有著不同層次的提升。

如果我今天想使用`List`這樣的`Collection`，我可以藉由命名，讓這個`List`充分表達出我想做的事情，但我又想限制這個物件的操作方法，那麼藉由一層物件的封裝，就可以達到這樣的效果。













