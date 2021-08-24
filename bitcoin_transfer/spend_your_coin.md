## Потратить монету {#spend-your-coin}

Итак, теперь, когда вы знаете, что такое **биткойн-адрес**, **ScriptPubKey**, **закрытый ключ** и **майнер**, вы совершите свою первую **транзакцию** вручную.

По мере прохождения этого урока вы будете добавлять код построчно, поскольку он представлен, чтобы создать метод, который оставит отзыв о книге в виде сообщения в стиле Twitter. Я _настоятельно рекомендую_ вам сначала следовать инструкциям в TestNet, а затем повторить их снова в основной сети Биткойн. 

Давайте начнем с рассмотрения **транзакции**, содержащей **TxOut**, который вы хотите потратить, как мы делали ранее:

Создайте новый **Console Project** \(&gt;.net45\)  и установите NuGet-пакет **QBitNinja.Client**.

Вы уже сами сгенерировали и записали закрытый ключ? Вы уже получили соответствующий биткойн-адрес и отправили туда средства? Если нет, не волнуйтесь, я быстро повторю, как вы можете это сделать:

```cs
// Replace this with Network.Main to do this on Bitcoin MainNet
var network = Network.TestNet;

var privateKey = new Key();
var bitcoinPrivateKey = privateKey.GetWif(network);
var address = bitcoinPrivateKey.GetAddress();

Console.WriteLine(bitcoinPrivateKey);
Console.WriteLine(address);
```

Note that we use the TestNet first, but you will probably do this on the MainNet as well, so you are going to spend real money! In any case, write down the **bitcoinPrivateKey** and the address! Send a few dollars of coins there and save the transaction ID \(you can find it in your wallet software or with a blockexplorer, like SmartBit for [MainNet](http://smartbit.com.au/) and [TestNet](https://testnet.smartbit.com.au/)).

Импортируйте свой закрытый ключ (замените строку «cN5Y ... K2RS» на свою):

```cs
var bitcoinPrivateKey = new BitcoinSecret("cN5YQMWV8y19ntovbsZSaeBxXaVPaK4n7vapp4V56CKx5LhrK2RS", Network.Testnet);
var network = bitcoinPrivateKey.Network;
var address = bitcoinPrivateKey.GetAddress();

Console.WriteLine(bitcoinPrivateKey); // cN5YQMWV8y19ntovbsZSaeBxXaVPaK4n7vapp4V56CKx5LhrK2RS
Console.WriteLine(address); // mkZzCmjAarnB31n5Ke6EZPbH64Cxexp3Jp
```

И, наконец, получите информацию о транзакции (замените «0acb ... b78a» на тот, который вы получили из программного обеспечения вашего кошелька или проводника блокчейна после того, как вы отправили монеты):

```cs
var client = new QBitNinjaClient(network);
var transactionId = uint256.Parse("0acb6e97b228b838049ffbd528571c5e3edd003f0ca8ef61940166dc3081b78a");
var transactionResponse = client.GetTransaction(transactionId).Result;

Console.WriteLine(transactionResponse.TransactionId); // 0acb6e97b228b838049ffbd528571c5e3edd003f0ca8ef61940166dc3081b78a
Console.WriteLine(transactionResponse.Block.Confirmations); // 91
```

Теперь у нас есть вся необходимая информация для создания наших транзакций. Основные вопросы: **откуда, куда и сколько?**

### Откуда?

В нашем случае мы хотим потратить второй аутпоинт. Вот как мы это выяснили:

```cs
var receivedCoins = transactionResponse.ReceivedCoins;
OutPoint outPointToSpend = null;
foreach (var coin in receivedCoins)
{
    if (coin.TxOut.ScriptPubKey == bitcoinPrivateKey.GetAddress(ScriptPubKeyType.Legacy).ScriptPubKey)
    {
        outPointToSpend = coin.Outpoint;
    }
}
if(outPointToSpend == null)
    throw new Exception("TxOut doesn't contain our ScriptPubKey");
Console.WriteLine("We want to spend {0}. outpoint:", outPointToSpend.N + 1);
```

Для оплаты вам необходимо указать этот аутпоинт в транзакции. Вы создаете транзакцию следующим образом:

```cs
var transaction = Transaction.Create(network);
transaction.Inputs.Add(new TxIn()
{
    PrevOut = outPointToSpend
});
```

### Куда?

Вы помните основные вопросы? ** Откуда, куда и сколько? **
Создание **TxIn** и добавление его к транзакции - это ответ на вопрос «откуда». 
Создание **TxOut** и добавление его к транзакции - это ответ на оставшиеся.

> The donation address of this book is: [1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB](https://www.smartbit.com.au/address/1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB)  
This money goes into Nicolas' "Coffee and Sushi Wallet" that will keep him fed and compliant while writing the rest of the book.  
If you succeed in completing this challenge on the MainNet you will be able to find your contribution among the **Hall of the Makers** on [http://n.bitcoin.ninja/](http://n.bitcoin.ninja/) \(ordered by generosity\).

Чтобы получить наш адрес MainNet:
```cs
var hallOfTheMakersAddress = new BitcoinPubKeyAddress("1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB", Network.Main);
```

Или, если Вы работаете в TestNet, отправьте монеты TestNet на любой адрес. Я использовал [mzp4No5cmCXjZUpf112B1XWsvWBfws5bbB] (https://testnet.smartbit.com.au/address/mzp4No5cmCXjZUpf112B1XWsvWBfws5bbB).

```cs
var hallOfTheMakersAddress = BitcoinAddress.Create("mzp4No5cmCXjZUpf112B1XWsvWBfws5bbB", Network.TestNet);
```

### Сколько?

Биткойн имеет [несколько единиц для использования] (https://en.bitcoin.it/wiki/Units), но есть три, о которых вам следует знать: биткойны, биты и сатоши. 1 биткойн (BTC) равен 1,000,000 бит, а 100 сатоши - это 1 бит. 1 сатоши (сат) - это самая маленькая единица в сети Биткойн.  

Если Вы хотите отправить **0,0004 BTC** (несколько долларов) из **неизрасходованного вывода**, который содержит **0,001 BTC**, Вам на самом деле придется потратить все это!
Как показано на диаграмме ниже, ваш **выход транзакции** указывает **0,0004 BTC** на [Hall of The Makers](http://n.bitcoin.ninja/) и **0,00053 BTC** обратно к Вам. 
Что будет с оставшимися **0,00007 BTC**? Это плата майнерам. 
Комиссия майнера побуждает майнеров добавить эту транзакцию в свой следующий блок. Чем выше комиссия майнера, тем больше у него мотивации для включения Вашей транзакции в следующий блок, а это означает, что Ваша транзакция будет подтверждена быстрее. Если Вы установите нулевую комиссию майнера, Ваша транзакция возможно никогда не будет подтверждена.

![](../assets/SpendTx.png)

```cs
transaction.Outputs.Add(Money.Coins(0.0004m), hallOfTheMakersAddress.ScriptPubKey);
// Send the change back
transaction.Outputs.Add(new Money(0.00053m, MoneyUnit.BTC), bitcoinPrivateKey.ScriptPubKey);
```

Здесь мы можем сделать некоторую тонкую настройку, давайте посчитаем изменение на основе комиссии майнера.  

```cs
// How much you want to spend
var hallOfTheMakersAmount = new Money(0.0004m, MoneyUnit.BTC);

// How much miner fee you want to pay
/* Depending on the market price and
 * the currently advised mining fee,
 * you may consider to increase or decrease it.
 */
var minerFee = new Money(0.00007m, MoneyUnit.BTC);

// How much you want to get back as change
var txInAmount = (Money)receivedCoins[(int) outPointToSpend.N].Amount;
var changeAmount = txInAmount - hallOfTheMakersAmount - minerFee;
```

Давайте вместо этого будем использовать наши вычисленные значения для нашего TxOuts:

```cs
transaction.Outputs.Add(hallOfTheMakersAmount, hallOfTheMakersAddress.ScriptPubKey);
// Send the change back
transaction.Outputs.Add(changeAmount, bitcoinPrivateKey.ScriptPubKey);
```

### Сообщение в блокчейне

А теперь добавьте свой личный отзыв! Он должен быть меньше или равен 80 байтам, иначе ваша транзакция будет отклонена.  
Это сообщение вместе с вашей транзакцией появится \(после подтверждения транзакции\) в [Hall of The Makers](http://n.bitcoin.ninja/)! :)

```cs
var message = "Long live NBitcoin and its makers!";
var bytes = Encoding.UTF8.GetBytes(message);
transaction.Outputs.Add(Money.Zero, TxNullDataTemplate.Instance.GenerateScriptPubKey(bytes));
```

### Резюме
Подводя итог, давайте посмотрим на всю транзакцию, прежде чем мы ее подпишем: 
У нас есть 3 **TxOut**, 2 с **количеством**, 1 без **количества** \(которое содержит сообщение\). Вы можете заметить различия между **scriptPubKey** в "обычных" **TxOut** и **scriptPubKey** в **TxOut** в сообщении:

```json
{
  "hash": "eeffd48b317e7afa626145dffc5a6e851f320aa8bb090b5cd78a9d2440245067",
  "ver": 1,
  "vin_sz": 1,
  "vout_sz": 3,
  "lock_time": 0,
  "size": 164,
  "in": [
    {
      "prev_out": {
        "hash": "0acb6e97b228b838049ffbd528571c5e3edd003f0ca8ef61940166dc3081b78a",
        "n": 0
      },
      "scriptSig": ""
    }
  ],
  "out": [
    {
      "value": "0.00040000",
      "scriptPubKey": "OP_DUP OP_HASH160 d3a689bc36464b9d74e1721fd321d4686eae594e OP_EQUALVERIFY OP_CHECKSIG"
    },
    {
      "value": "0.00053000",
      "scriptPubKey": "OP_DUP OP_HASH160 376b786582a3423bcdda4517ea87f0a7e862f27b OP_EQUALVERIFY OP_CHECKSIG"
    },
    {
      "value": "0.00000000",
      "scriptPubKey": "OP_RETURN 4c6f6e67206c697665204e426974636f696e20616e6420697473206d616b65727321"
    }
  ]
}
```

Присмотритесь к **TxIn**. У нас есть **prev\_out** и **scriptSig**
**Упражнение:** попробуйте выяснить, каким будет **scriptSig** и как получить его в нашем коде, прежде чем читать дальше!

Давайте проверим **хэш** из **prev\_out** в блокэксплорере TestNet: [prev\_out tx details](https://testnet.smartbit.com.au/tx/0acb6e97b228b838049ffbd528571c5e3edd003f0ca8ef61940166dc3081b78a).  Как видите, на наш адрес было переведено 0,001 BTC.

In **prev\_out** **n** is 0. Since we are indexing from 0, this means that we want to spend the first output of the transaction (the second one is the 1.0989548 BTC change from the transaction). 
В **prev \ _out** **n** равно 0. Поскольку мы индексируем с 0, это означает, что мы хотим потратить первый вывод транзакции (второй - изменение 1.0989548 BTC из транзакции).

### Подпишите транзакцию

Теперь, когда мы создали транзакцию, мы должны ее подписать. Другими словами, вам нужно будет доказать, что вы владеете TxOut, на который вы ссылались во входных данных.

Подписание может быть [сложным](https://en.bitcoin.it/w/images/en/7/70/Bitcoin_OpCheckSig_InDetail.png), но мы сделаем это проще.

Сначала давайте вернемся к **scriptSig** из **in** и как мы можем получить его из кода. У нас есть два варианта заполнения ScriptSig ключом ScriptPubKey нашего адреса:

```cs
// Get it from the public address
var address = BitcoinAddress.Create("mkZzCmjAarnB31n5Ke6EZPbH64Cxexp3Jp", Network.TestNet);
transaction.Inputs[0].ScriptSig = address.ScriptPubKey;

// OR we can also use the private key 
var bitcoinPrivateKey = new BitcoinSecret("cN5YQMWV8y19ntovbsZSaeBxXaVPaK4n7vapp4V56CKx5LhrK2RS", Network.TestNet);
transaction.Inputs[0].ScriptSig =  bitcoinPrivateKey.ScriptPubKey;
```

Затем необходимо предоставить Ваш закрытый ключ, чтобы подписать транзакцию:

```cs
transaction.Sign(bitcoinPrivateKey, receivedCoins.ToArray());
```
После этой команды свойство ScriptSig будет заменено подписью, что сделает транзакцию подписанной. 

Вы можете проверить нашу транзакцию [TestNet в проводнике блокчейна](https://testnet.smartbit.com.au/tx/eeffd48b317e7afa626145dffc5a6e851f320aa8bb090b5cd78a9d2440245067).

### Propagate your transactions

Congratulations, you have signed your first transaction! Your transaction is ready to roll! All that is left is to propagate it to the network so the miners can see it.

#### With QBitNinja:

```cs
BroadcastResponse broadcastResponse = client.Broadcast(transaction).Result;

if (!broadcastResponse.Success)
{
    Console.Error.WriteLine("ErrorCode: " + broadcastResponse.Error.ErrorCode);
    Console.Error.WriteLine("Error message: " + broadcastResponse.Error.Reason);
}
else
{
    Console.WriteLine("Успех! Вы можете проверить хэш транзакции в любом проводнике блоков:");
    Console.WriteLine(transaction.GetHash());
}
```

#### With your own Bitcoin Core:

```cs
using (var node = Node.ConnectToLocal(network)) //Подключиться к узлу
{
    node.VersionHandshake(); //Say hello
                             //Advertize your transaction (send just the hash)
    node.SendMessage(new InvPayload(InventoryType.MSG_TX, transaction.GetHash()));
    //Send it
    node.SendMessage(new TxPayload(transaction));
    Thread.Sleep(500); //Wait a bit
}
```

The **using** code block will take care of closing the connection to the node. That's it!

You can also connect directly to the Bitcoin network, however I advise you to connect to your own trusted node as it is faster and easier.

## Need more practice?

YouTube: [How to make your first transaction with NBitcoin](https://www.youtube.com/watch?v=X4ZwRWIF49w)  
CodeProject: [Create a Bitcoin transaction by hand.](http://www.codeproject.com/Articles/1151054/Create-a-Bitcoin-transaction-by-hand)  
CodeProject: [Build your own Bitcoin wallet](https://www.codeproject.com/Articles/1115639/Build-your-own-Bitcoin-wallet)

