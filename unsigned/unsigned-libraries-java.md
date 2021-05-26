Решения для работы с Unsigned числами + исследования
====================================================

Самый универсальный способ работы с целочисленными примитивными типами — это использование специализированных классов-обёрток. Под капотом будут находиться обычные типы Java, а обёртка берёт на себя все функции по их обработке.

Одного, универсального решения, конечно, не существует. На фоне остальных выделяется Google Guava, но на практике разработчик скорее всего выберет в качестве примера одно из доступных решений, и сам напишет то, что удовлетворяет его требованиям.


|              | Google Guava | Apache Axis | jOOU | OPC Foundation | JNIWrapper | YSTT/Kaitai |
|     :---:    |     :---:    |     :---:      |     :---:      |     :---:      |     :---:      |     :---:      |
| Создание     |	***	        | *** |	*** |	*** |	*** |	*** |
| Передача     |	***	        | *** |	*** |	*** |	*** |	- |
| Обработка    |	***	        | * |	* |	** |	- |	- |
| Сохранение   |	***         |	*** |	*** |	*** |	*** |	*** |
| Память       |	**	        | * |	* |	** |	* |	*** |
| Процессор    |	**          | 	** |	** |	** |	** |	*** |
| Автономность |	***         |	** |	** |	*** |	*** |	* |

Давайте рассмотрим, что интересного можно найти в реализациях от разных авторов.

Google Guava
------------

Это полноценная библиотека для поддержки беззнаковых типов. Имеет в арсенале классы-обёртки, `UnsignedInteger, UnsignedLong`, и вспомогательные классы для всех примитивных типов.

UnsignedInteger может создаваться из переменных типов `int, long, BigInteger, String`, а так же из другого беззнакового `int`. Под капотом находится переменная типа `int`. Методы:

*	plus
*	minus
*	times
*	divideBy
*	mod

UnsignedLong работает по такому же принципу, но под капотом переменная типа long.

Классы `UnsignedBytes, InsignedInts, UnsignedLongs` функционально очень похожи. Рассмотрим методы класса `UnsignedInts`:

*	checkedCast (long -> int), бросает ошибку
*	saturatedCast (long -> int), преобразует тип так близко, насколько это возможно
*	compare
*	min
*	max
*	join – объединяет массив примитивного типа в строку
*	sort – сортирует массив примитивного типа
*	sortDescending
*	divide
*	remainder
*	parseUnsigned…
*	decode (Hex String)


Apache Axis
-----------

Беззнаковое число для всех типов UnsignedByte, UnsignedShort, UnsignedInt хранится в переменной типа Long. UnsignedLong сделан на основе BigInteger.

Вариантов создания этих объектов очень много, в качестве аргументов конструктора можно использовать и примитивные типы, и BigInteger и даже строки.

Эти классы используются только для передачи данных, в них нет поддержки беззнаковой арифметики. Единственное что они позволяют, это выполнять беззнаковое сравнение.


jOOU
----

Классы-обёртки: `UByte, UInteger, ULong, UShort, UNumber`.
Вспомогательные классы: `UMath, Unsigned`.

Классы-обёртки кэшируют определённые диапазоны значений. Например, UByte – все возможные 256 чисел. В этих классах много перегруженных методов valueOf, так же есть перегруженные методы add и subtract.

Производительность достигается за счёт большего потребления памяти:

*	UByte: short
*	UShort: int
*	UInteger: long
*	ULong: BigInteger

Класс UMath содержит перегруженные методы max и min.

Класс Unsigned содержим фабричные методы для создания беззнаковых типов.


OPC Foundation UA JAVA Legacy
-----------------------------

На первый взгляд кажется, что потребление памяти будет избыточным:

* UnsignedByte: int
* UnsignedShort: int
* UnsignedInteger: int
* Unsigned long: long

Однако, за счёт кэширования это не должно быть сильно заметно.

Классы имеют несколько типов конструкторов, методы: `valueOf, max, min, inc, dec, add, subtract`.


DIS-Java-VRML
-------------

Просто набор классов-обёрток, не несущих какой-либо дополнительной функциональности. Проект очень старый, не нашлось даже исходников, лишь JavaDoc.


JNIWrapper 
----------

Это платный продукт, в котором есть поддержка разнообразнейших типов, включая: `UInt, UInt16, UInt32, UInt64, UInt8, UShortInt`.

Складывается впечатление, что эти классы не очень дружелюбны к памяти. По крайней мере, метод getValue для всех этих типов возвращает long. 

Из необычного – методы для записи в DataBuffer.


Yoda Stories Translation Tool / Kaitai Struct
---------------------------------------------

*Здесь описано решение, при котором беззнаковые типы сопровождаются только на этапах чтения и записи.*

Утилита Yoda Stories Translation Tool изначально использовала следующее решение: класс Dump, хранящий байтовый массив, и обеспечивающий чтение/запись чисел различной разрядности и порядка байтов, а также сохранение и загрузку массива.

Впоследствии часть кода была заменена более прогрессивным решением.

Язык Kaitai Struct описывает структуру различных бинарных форматов: архивов, различных мультимедиа файлов, файловых систем, баз данных, исполняемых файлов, сетевых протоколов, и даже файлов данных из различных игр.

Для разработки KSY файлов есть даже свои IDE. После описания структуры бинарного файла, на её основе можно скомпилировать код для чтения этой структуры, причём, поддерживается порядка 12-ти языков, включая и Java.

Этот код представляет собой иерархию классов, описывающих структуру бинарного файла. Чтение данных производится последовательно. В каждом классе есть метод _read(), который забирает свою порцию данных для инициализации своих полей.

Дополнительно следует подключить библиотеку, отвечающую за чтение из файлов или байтовых массивов. KaitaiStream имеет две реализации, для последовательного и произвольного доступа к бинарным данным.

В названии всех методов используется информация о типе данных:

*	Со знаком или без: S/U
*	Разрядность исходных данных: 1,2,4,8
*	Порядок байтов: be, le

Пример:

```
ByteBuffer bb;

public byte readS1() { // read signed byte
    return bb.get();
}

public int readS4be() { // read signed int with BigEndian byte order
    bb.order(ByteOrder.BIG_ENDIAN);
    return bb.getInt();
}

public long readU4le() { // read unsigned int with LittleEndian order
    bb.order(ByteOrder.LITTLE_ENDIAN);
    return bb.getInt() & 0xffffffffL;
}
```

Числа без знака занимают в памяти в два раза больше места, но зато к ним можно применять любые операции Java без ограничений. А что насчёт long?

```
public long readU8le() { // read “unsigned” long with LittleEndian order 
    return readS8le();
}
```

Переиспользуется метод для чтения long со знаком, соответственно, если понадобится менять это значение, то придётся дополнительно потрудиться.

Так же есть методы для чтения чисел с плавающей запятой, байтовых массивов, битов и строк. Поддерживается сравнение массивов, а также поиск минимума и максимума.

Поскольку работа Yoda Stories Translation Tool подразумевает изменение и сохранение данных, эта функциональность была доработана.

*	KaitaiInputStream
    *	ByteBufferKaitaiInputStream
    *	RandomAccessKaitaiInputStream
*	KaitaiOutputStream
    *	ByteBufferKaitaiOutputStream


KaitaiOutputStream это зеркальная реализация KaitaiInputStream для записи бинарных данных в файл, байтовый массив или ByteBuffer.

Пример методов:

```
public void writeS4be(int value) {
    bb.order(ByteOrder.BIG_ENDIAN);
    bb.putInt(value);
}

public void writeU4le(int value) {
    bb.order(ByteOrder.LITTLE_ENDIAN);
    bb.putInt(value);
}

public void writeU4le(long value) {
    writeU4le((int) value);
}
```

Выполняется простое приведение типа, и оно не вызывает никаких проблем, если в long хранится значение, не выходящее за пределы int. Проверки на выход за пределы области допустимых значений нигде не выполняются.

В каждом классе, описывающем бинарный файл в дополнение к методу _read реализован полностью зеркальный метод _write.



### Хранение данных в виде массива байтов

Здесь будет показан пример хранения данных в виде массива байтов. Этот способ нашёл своё применение в утилите Yoda Stories Translation Tool.

Он привлекателен тем, что занимает ровно столько памяти, сколько занимает бинарный файл.

Класс Dump содержит в себе не только массив, но и подмножество методов для доступа к его элементам.

Есть возможность получать данные с различным порядком байтов:

```
public enum ByteOrder {
    BIG_ENDIAN,
    LITTLE_ENDIAN
}
```

Конструкторы позволяют загрузить данные из файлов, или создать буфер необходимой вместимости. Поскольку в большинстве случаев данные читаются последовательно, то реализована поддержка указателя и базового смещения. При каждом чтении данных указатель сдвигается на размер прочитанных данных. Методы для чтения: 

```
getByte
getWord
getLongWord
getChar
getString
getBoolean
getArray
```

К качеству и производительности кода может быть много претензий, но, по крайней мере вычисления наглядно показывают, что происходит:

```
public int getWord() {
  int result;
  switch (byteOrder) {
    case LITTLE_ENDIAN:
    result = Byte.toUnsignedInt(dump[index]);
    index++;
    result += Byte.toUnsignedInt(dump[index]) * 256;
    index++;
    break;
  default:
    result = Byte.toUnsignedInt(dump[index]) * 256;
    index++;
    result += Byte.toUnsignedInt(dump[index]);
    index++;
    break;
  }
  return result;
}
```

Существуют аналогичные методы для изменения значений: `setByte`, …

Другие методы:

*	writeHexDump
*	erase
*	get/setOffset
*	get/setIndex
*	insertEmptyArea
*	deleteArea
*	findAddress
*	findValueAddressByMask
*	findAddressWithMask
*	get/setCharset.

Кодировка необходима, если извлекаются строковые значения. По умолчанию это Cp2152.

Пожалуй, это всё, что хотелось рассказать про работу с беззнаковыми типами в Java. Если у вас есть вопросы, или вы заметили какие-то неточности или недочёты, то обязательно напишите мне. Благодарю.