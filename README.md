# ДЗ №1 К занятию “Обзорная лекция - смарт-контракты”
## Выполнил работу Миленин Иван (M33351)

### Задание 1
***
https://github.com/gnosis/MultiSigWallet/blob/master/contracts/MultiSigWallet.sol - сделать, чтобы с баланса multisig-контракта за одну транзакцию не могло бы уйти больше, чем 66 ETH
***
```
      *  Constants
      */
     uint constant public MAX_OWNER_COUNT = 50;
+    uint constant public MAX_TRANSACTION_SIZE = 66;
```

```
         _;
     }

+    modifier validTransactionValue(uint transactionSize) {
+        require(transactionSize <= MAX_TRANSACTION_SIZE);
+        _;
+    }
+
```

```
 /// @return Returns transaction ID.
     function submitTransaction(address destination, uint value, bytes data)
         public
+        validTransactionValue(value)
         returns (uint transactionId)
     {
```

#### Объяснение:
В данном задании мы воспользуемся созданием `modifier` функции, которая будет проверять, не превысило ли полученное ей значение `MAX_TRANSACTION_SIZE = 66`. Далее добавляем `modifier validTransactionValue` в соответствующую функцию `function submitTransaction`.

### Задание 2
***
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/f2112be4d8e2b8798f789b948f2a7625b2350fe7/contracts/token/ERC20/ERC20.sol - сделать, чтобы токен не мог бы быть transferred по субботам
***

```
     string private _name;
     string private _symbol;
+    uint constant public SECOND_IN_DAY = 24 * 60 * 60;
+    uint constant public BLOCK_DAY_WEEK = 6;
```

```
     function _transfer(address sender, address recipient, uint256 amount) internal virtual {
         require(sender != address(0), "ERC20: transfer from the zero address");
         require(recipient != address(0), "ERC20: transfer to the zero address");
+        require(getDayOfWeek() != BLOCK_DAY_WEEK, "ERC20: transfer at saturday");
```

```
+    function getDayOfWeek() public returns (uint) {
+        uint today = now;
+        today = ((today / SECOND_IN_DAY) + 4) % 7;
+        return today;
+    }
+
```

#### Объяснение:
У Solidity есть возможность вернуть `unix timestamp` благодаря `now`. Засчет нему и несложным вычислениям мы можем получить текущий день недели *(где 1 - Пн, 2 - Вт, ... , 7 - Вс)*. Для этого создадим функцию `function getDayOfWeek()`.

В основной метод по переводу кладем проверку на то, что текущий день недели не равен субботе, а именно значению `BLOCK_DAY_WEEK`.

### Задание 3

***
https://github.com/mixbytes/solidity/blob/076551041c420b355ebab40c24442ccc7be7a14a/contracts/token/DividendToken.sol - сделать чтобы платеж в ETH принимался только специальной функцией, принимающей помимо ETH еще комментарий к платежу (bytes[32]). Простая отправка ETH в контракт запрещена
***

```
     event PayHangingDividend(uint256 amount) ;
-    event Deposit(address indexed sender, uint256 value);
+    event Deposit(address indexed sender, uint256 value, bytes[32] text);
```

```
     }

+    bytes[32] constant private EMPTY_TEXT = "";
+
     constructor () public
```

```
-    function() external payable {
-        if (msg.value > 0) {
-            emit Deposit(msg.sender, msg.value);
+    function(bytes[32] text) external payable {
+        if (msg.value > 0 && text != EMPTY_TEXT) {
+            emit Deposit(msg.sender, msg.value, text);
             m_totalDividends = m_totalDividends.add(msg.value);
         }
     }
```

#### Объяснение:
Проанализировав код, я выяснил, что платеж производится засчет ` event Deposit`. Тогда необходимо добавить к нему дополнительно переменную `bytes[32] text`, в которой будет находится сообщение.

Также нужно изменить нашу `payable` - функцию (которая теперь должна получать на вход не пустой текст *(хотя точно не знаю, считается ли пустое сообщение - сообщением. Будем считать, что нет)*).