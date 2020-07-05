# **函式**
## **簡短**
**函式的首要準則，就是簡短。第二項準則，就是比第一項簡短函式還要簡短。**

雖然沒有任何一項證據或是文獻，說明簡短函式會比較好，但就Uncle Bob的開發經驗，提供了這項準則。

### 區塊(Blocks)和縮排(Indenting)
`if`、`else`、`while`內都應該只有一行，除了維持函式簡短，也應該用良好的命名表達意圖，讓它添加出類似說明文件的價值。同時也意味著，不要寫出大量的巢狀結構，縮排應該只有一兩層。

## **只做一件事情**
**函式應該做一件事情。它們應該把這件事做好。而且它們也只該做這件事情**

什麼叫只做一件事情呢 ? 只要函式內部做的事情，都處於函式名稱下的抽象概念，就叫做一件事，好比這段程式碼 :
```
public static String renderPageWithSetupsAndTeardowns(
 PageData pageData, boolean isSuite) throws Exception {
    if (isTestPage(pageData))
        includeSetupAndTeardownPages(pageData, isSuite);
    return pageData.getHtml();
```
看起來有好幾個步驟，但實際上，就是所謂的只做一件事，全部的事情，都符合函式名稱下的抽象概念。
### 一層抽象概念
抽象概念也有分層上的關係，想像一下，如果想要產生一個頁面，我們可能會有一個方法叫做`getHtml()`，這是高層次的抽象概念，而為了做這件事情，或許會去某個路徑下撈資料，那就有個中層次抽象概念的程式碼 : `String pagePathName = PathParser.render(pagePath);`，而低層次則會像是單純的字串拼接 : `.append("\n")`。除了要讓函式只做一件事，也要盡力讓內部都處於同一層抽象概念，並且符合降層原則 ( 高>中>
低 )。

### 函式的段落
另外，如果函式內部可以拆分成好幾個區塊 ( ex.宣告區、檢核區、邏輯區...... )，很明顯地就超過一件事情。

## **Switch敘述 ( 一連串的if/else )**
```
public Money calculatePay(Employee e) throws InvalidEmployeeType {
    switch (e.type) {
        case COMMISSIONED:
            return calculateCommissionedPay(e);
        case HOURLY:
            return calculateHourlyPay(e);
        case SALARIED:
            return calculateSalariedPay(e);
        default:
            throw new InvalidEmployeeType(e.type);
    }
}
```
這函式有好幾個問題 :
* 太過冗長，加入新的職員型態會更冗長。
* 做超過一件事情 ( 單一職責原則 )。
* 當新型態加入，函式就必須改變，違反開放閉合原則。

然而最大的問題，並不是這些，而是當`Class`內有多個這種方法 ( 好比`isPayday(Employee e)`、`deliverPay(Employee e)` )，整個`Class`就會充斥著switch敘述。

而解決方法，就是將switch敘述埋在抽象工廠 ( Abstract Factory ) 底下 :
```
public abstract class Employee {
    public abstract boolean isPayday();
    public abstract Money calculatePay();
    public abstract void deliverPay(Money pay);
}
-----------------
public interface EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType;
}
-----------------
public class EmployeeFactoryImpl implements EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
        switch (r.type) {
            case COMMISSIONED:
                return new CommissionedEmployee(r) ;
            case HOURLY:
                return new HourlyEmployee(r);
            case SALARIED:
                return new SalariedEmploye(r);
            default:
                throw new InvalidEmployeeType(r.type);
        }
    }
}
```
## **使用具備描述能力的名稱**
盡量讓命名都有著一致性的名詞、動詞，因為當依序看到這些要執行的函式，就會像是看到一個有著起承轉合的故事 :
```
includeSetupAndTeardownPages()
includeSetupPages()
includeSuiteSetupPage()
includeSetupPage()
```
## **函式的參數**
參數基本上越少越好，零個是最好的，而如果要使用到三個以上的參數，建議慎重思考一下，是否有這樣的必要。

參數的具有透露概念的能力，比起有參數的函式，閱讀沒有參數的函式，會讓人更專心注意在函式名稱上，更容易搞清楚函式的意圖，可以想想如果今天把`StringBuilder`當作參數傳到函式內，除了閱讀函式名稱外，可能還要花心力看為何要傳入`StringBuilder`，更嚴重的，這個傳入參數可能還與函式在不同層的抽象概念。

從測試角度來看，越多的參數就有越多種可能的測試案例，導致測試的困難。

### 單一參數的常見形式
通常分成以下三種 :
* 會問與這個參數有關的問題，像是`boolean fileExists("MyFile")`。
* 對某個參數進行轉換，並且回傳，建議一定要有回傳值，即使回傳的物件和傳入的物件相同，避免引起困惑 :
    ```
    StringBuffer transform(StringBuffer in) //better
    void transform-(StringBuffer out)
    ```
* 事件型的函式，像是`void passwordAttempFailedNtimes(int attempts)`，通常用在修改系統的某種狀態。

### 旗標參數
使用`boolean`當作參數的一種，通常代表函示要做**兩件事情**，因此建議將這函式，拆成兩個函式。

### 兩個參數的函式
`writeField(name)`、`writeField(outputStream, name)`，看看這兩個函式，第一個函式應該會比第二個還好理解，而第二個函式在反覆看之後，我們可能會忽略掉`outputStream`這個參數，畢竟看起來它跟函式名稱要做的事比較沒關係，然而，忽略的地方，往往就是出現bug的地方。

不過當然也有合理的情況，好比`Point`類別，一個座標上本來就該有x、y軸，然而以這案例來講，這兩個參數是**有序元件組成的單一值**，與前者不同。

另一種搞混的情況則是這種`assertEquals(expected, actual)`，其實很容易搞混`expected`、`actual`的順序。

這邊舉了幾個，使用兩個參數可能會遇到的情況，並不是說使用兩個參數絕對不好，但如果能替換成一個參數，就能避免這些問題。像是把`outputStream`放在成員變數裡，或者另外分出一個類別`FileWriter`來處理這件事情。

### 三個參數的函式
三個參數的問題，當然會更加嚴重，不過這邊提了一個例子 :
```
assertEquals(1.0, amount, .001)
```
這是單純的檢查數字精確度的問題，相對起來就比較還好。
### 物件型態的參數
```
Circle makeCircle(double x, double y, double radius);
Circle makeCircle(Point center, double radius); //beter
```
變數`x`、變數`y`是同屬於某個概念，那包成一個物件當作參數會比較恰當些。
### 動詞和關鍵字
除了上一章解釋的，給函式一個好名稱之外，另一個是關鍵字形式命名 :
```
assertExpectedEqualsActual(expected, actual)
```
把參數名稱放到命名內，這就減輕記住順序的負擔。
## 要無副作用
有時函式看起來只做一件事情，但其實在背後，偷跑了一些東西，好比更改類別的某個變數，或是偷將它傳給其他類別去操作 :
```
public class UserValidator {
    private Cryptographer cryptographer;
    public boolean checkPassword(String userName, String password) {
        User user = UserGateway.findByName(userName);
        if (user != User.NULL) {
            String codedPhrase = user.getPhraseEncodedByPassword();
            String phrase = cryptographer.decrypt(codedPhrase, password);
            if ("Valid Password".equals(phrase)) {
                Session.initialize();
                return true;
            }
        }
    return false;
    }
}
```
以這段程式碼來說，有個陷阱藏在`Session.initialize();`，它做了一些我們不清楚的事情，很明顯的這函式違反了只做一件事情的原則，如果不打算再拆出一個函式，那就把名稱改成`checkPasswordAndInitializeSession`吧。
## **指令和查詢分離**
函式應該要能夠回答問題，或者做某些事情，來看看這個例子 :
```
public boolean set(String attribute, String value);
if (set("username", "unclebob"))...
```
在`if()`裡面其實有點詭異，是想表達是否`username`已被設定成`Unclebob`，還是`username`是否有被成功設定成`Unclebob` ? 雖然可以利用命名改善這個問題，但還是建議將倆著拆分開來 :
```
if (attributeExists("username")) {
    setAttribute("username", "unclebob");
    ... 
}
```
## **使用例外處理取代回傳錯誤代碼**
當使用錯誤代碼時，通常會引發冗長的巢狀結構，改用`try-catch`會較為簡潔 :
```
if (deletePage(page) == E_OK) {
    if (registry.deleteReference(page.name) == E_OK) {
        if (configKeys.deleteKey(page.name.makeKey()) == E_OK){
            logger.log("page deleted");
        } else {
            logger.log("configKey not deleted");
        }
    } else {
        logger.log("deleteReference from registry failed");
    }
} else {
    logger.log("delete failed");
    return E_ERROR;
}
/////////////
try {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
} catch (Exception e) {
    logger.log(e.getMessage());
}
```
然而更進一步地，應該**把錯誤處理當成一件事情** :
```
 public void delete(Page page) {
    try {
        deletePageAndAllReferences(page);
    } catch (Exception e) {
        logError(e);
    }
}

private void deletePageAndAllReferences(Page page) throws Exception {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
}
```
## **不要重複自己**
許多準則和慣例，目的都是為了消除重複的程式碼而衍生。像是資料庫的Codd's normal forms ( 柯德正規法 ) 用來消除資料的重複。或是物件導向程式設計、結構化程式設計、剖面導向程式設計、元件導向程式設計，這些都是為了去除重複程式碼。

## **結構化程式設計**
Edsger Dijkstra提出結構化程式設計，認為每個函式及函式區塊裡，只有一個進入點及離開點，也就是說，只有一個`return`敘述，迴圈內不得有`break`跟`continue`敘述，更不該有`goto`。

不過書中認為，如果保持函式的短小，偶爾出現這些敘述其實沒什麼不好，畢竟短小的函式有單一進出入點助益也不大，但如果是大型的函式區塊，這樣的規則就相當有助益。

## **尾加碎碎念**
chapter 3 是我第二篇完成的Clean Code讀書紀錄，原因單純只是chapter 2、chapter 3分開在兩個電腦寫，每次寫完一個段落都覺得好累啊，連git都懶得用，所以造成現在的情況了 ( 笑。而在做筆記的過程中，常常會覺得每個地方都滿重要的，就落落長的把每一段都做了一些紀錄，我猜在看這篇的人，應該也有這個想法，怎麼一篇摘要會需要讀這麼久XD。

對目前的我來說，這邊的概念算是全新的，從開始工作到現在也才快一年，寫程式的經驗相當不足，很坦白的說，很多時候只是經過了兩三個月，回頭看看自己之前寫得程式，常會覺得慘不忍睹，想想就對於之後要維護系統的人感到愧疚啊~

在記筆記的同時，腦中突然想到一個問題，書上大部分的範例程式，全都是在使用Java來編寫，這也當然，畢竟大部分提到的概念都是運用物件導向程式的特性所提出來的，那如果今天是寫JavaScript呢 ? 雖然公司是使用舊時代的技術jsp來編寫網頁，而不是使用新的前端框架，但就算是舊技術，應該還是有些編寫的原則，可以讓程式碼變得好閱讀好維護吧 ? 可是大部分我看到的情況都不是如此，而且要把這邊看到的概念移植用在JavaScript上，總覺得不太可能，不知道JavaScript有沒有一些開發上的準則可以參考。

### 不清楚的部分

這章有個名詞不太清楚 : 輸出型參數 ( output arguments )，即使回頭去看了原文版的電子書、或是在google上search相關的資料，還是沒辦法精確理解書上想表達的意思，有一些自己的猜測的想法，不過自己想想如果不確定的東西，還是別放在摘要的部分，放在心得的區塊也許會比較適當一點。

書上的所舉的例子，是`appendFooter(s)`這個`function`，想像我們要產出一個`html footer`，而`s`屬於要產出的一部分，這就是輸出型參數的意思，我們可能會把某樣東西，`append`到`s`後面，或是`s`本身就是`footer`，總而言之，它會是`return`值的一部分，這是我對輸出型參數的理解。

然而可能會出現某種問題，到底是某樣東西放在`s`後面、還是`s`本身就是`footer` ? 我們需要去看`appendFooter`這個方法才能夠理解，那這可能就違背了Clean Code的精神，程式碼出現模稜兩可的情況，

而解決的方法，就是盡量避免這樣的參數出現，如果有這樣的需求，就直接去改動這個物件本身的內容 : `report.appendFooter()`。







  
















