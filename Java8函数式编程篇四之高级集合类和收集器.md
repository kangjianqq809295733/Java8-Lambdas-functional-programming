# 第5章 高级集合类和收集器

第3章只介绍了集合类的部分变化，事实上，`Java 8` 对集合类的改进不止这些。现在是时 候介绍一些高级主题了，包括新引入的 `Collector` 类。同时我还会为大家介绍方法引用，它可以帮助大家在 `Lambda` 表达式中轻松使用已有代码。编写大量使用集合类的代码时， 使用方法引用能让程序员获得丰厚的回报。本章还会涉及集合类的一些更高级的主题，比 如流中元素的顺序，以及一些有用的 `API`。

## 5.1 方法引用
读者可能已经发现，`Lambda` 表达式有一个常见的用法:`Lambda` 表达式经常调用参数。比如想得到艺术家的姓名，`Lambda` 的表达式如下:

```java
    artist -> artist.getName()
```

这种用法如此普遍，因此 `Java 8` 为其提供了一个简写语法，叫作方法引用，帮助程序员重用已有方法。用方法引用重写上面的 `Lambda` 表达式，代码如下:

```java
    Artist::getName
```

标准语法为 `Classname::methodName`。需要注意的是，虽然这是一个方法，但不需要在后面加括号，因为这里并不调用该方法。我们只是提供了和 `Lambda` 表达式等价的一种结构， 在需要时才会调用。凡是使用 `Lambda` 表达式的地方，就可以使用方法引用。

构造函数也有同样的缩写形式，如果你想使用 `Lambda` 表达式创建一个 `Artist` 对象，可能 会写出如下代码

```java
    (name, nationality) -> new Artist(name, nationality)
```
使用方法引用，上述代码可写为:

```java
    Artist::new
```

这段代码不仅比原来的代码短，而且更易阅读。`Artist::new` 立刻告诉程序员这是在创建 一个 `Artist` 对象，程序员无需看完整行代码就能弄明白代码的意图。另一个要注意的地方是方法引用自动支持多个参数，前提是选对了正确的函数接口。

还可以用这种方式创建数组，下面的代码创建了一个字符串型的数组:

```java
    String[]::new
```

从现在开始，我们将在合适的地方使用方法引用，因此读者很快会看到更多的例子。一开始探索 `Java 8` 时，有位朋友告诉我，方法引用看起来“就像在作弊”。他的意思是说，了解如何使用 `Lambda` 表达式让代码像数据一样在对象间传递之后，这种直接引用方法的方式就像“作弊”。

放心，这不是在作弊。读者只要记住，每次写出形如 `x -> a.foo(x)`的`Lambda`表达式时， 和直接调用方法 `foo` 是一样的。方法引用只不过是基于这样的事实，提供了一种简短的语法而已。

```
x -> a.foo(x) <==> A::foo
```

## 5.2 元素顺序
另外一个尚未提及的关于集合类的内容是流中的元素以何种顺序排列。读者可能知道，一 些集合类型中的元素是按顺序排列的，比如 `List`;而另一些则是无序的，比如 `HashSet`。 增加了流操作后，顺序问题变得更加复杂。

直观上看，流是有序的，因为流中的元素都是按顺序处理的。这种顺序称为出现顺序。出现顺序的定义依赖于数据源和对流的操作。

在一个有序集合中创建一个流时，流中的元素就按出现顺序排列，因此，如下代码总是可以通过。

- 顺序测试永远通过

```java
    @Test
    public  void testStreamSort(){
        List<Integer> numbers = Arrays.asList(1,2,3,4);
        List<Integer> sameOrder = numbers.stream()
                                            .collect(Collectors.toList());
        assertEquals(numbers,sameOrder);

    }
```
如果集合本身就是无序的，由此生成的流也是无序的。`HashSet` 就是一种无序的集合，因此不能保证如下所示的程序每次都通过。

```java
    @Test
    public  void testHashSetOrder(){
        Set<Integer> numbers = new HashSet<>(Arrays.asList(4,3,2,1));
        List<Integer> sameOrder = numbers.stream()
                                            .collect(Collectors.toList());
        assertEquals(Arrays.asList(4,3,2,1), sameOrder);
    }
```

流的目的不仅是在集合类之间做转换，而且同时提供了一组处理数据的通用操作。有些集合本身是无序的，但这些操作有时会产生顺序，试看如下代码。

```java
    @Test
    public  void testStreamSort1(){
        Set<Integer> numbers = new HashSet<>(Arrays.asList(4,3,2,1));
        List<Integer> stillOrdered = numbers.stream()
                                            .sorted()
                                            .collect(Collectors.toList());
        assertEquals(Arrays.asList(1,2,3,4),stillOrdered);
    }
```

一些中间操作会产生顺序，比如对值做映射时，映射后的值是有序的，这种顺序就会保留下来。如果进来的流是无序的，出去的流也是无序的。看如下代码，我们只能断言`HashSet `中含有某元素，但对其顺序不能作出任何假设，因为 `HashSet` 是无序的，使用了映射操作后，得到的集合仍然是无序的。

```java
    @Test
    public  void testStreamMapSort(){
        List<Integer> numbers = Arrays.asList(1,2,3,4);
        List<Integer> stillOrdered = numbers.stream()
                                            .map(x->x+1)
                                            .collect(Collectors.toList());
        //顺序得到了保留
        assertEquals(Arrays.asList(2, 3, 4, 5), stillOrdered);
        Set<Integer> unordered = new HashSet<>(numbers);
        List<Integer> stillUnordered = unordered.stream()
                                                .map(x->x+1)
                                                .collect(Collectors.toList());
        // 顺序得不到保证
        assertThat(stillUnordered, hasItem(2));
        assertThat(stillUnordered, hasItem(3));
        assertThat(stillUnordered, hasItem(4));
        assertThat(stillUnordered, hasItem(5));

    }
```

一些操作在有序的流上开销更大，调用 `unordered` 方法消除这种顺序就能解决该问题。大多数操作都是在有序流上效率更高，比如 `filter、map` 和 `reduce` 等。

这会带来一些意想不到的结果，比如使用并行流时，`forEach` 方法不能保证元素是 按顺序处理的(第 6 章会详细讨论这些内容)。如果需要保证按顺序处理，应该使用 `forEachOrdered` 方法，它是你的朋友。

## 5.3 使用收集器
前面我们使用过 `collect(toList())`，在流中生成列表。显然，`List` 是能想到的从流中生成的最自然的数据结构，但是有时人们还希望从流生成其他值，比如 `Map` 或 `Set`，或者你希望定制一个类将你想要的东西抽象出来。

前面已经讲过，仅凭流上方法的签名，就能判断出这是否是一个及早求值的操作。`reduce` 操作就是一个很好的例子，但有时人们希望能做得更多。

这就是收集器，一种通用的、从流生成复杂值的结构。只要将它传给 `collect` 方法，所有 的流就都可以使用它了。

标准类库已经提供了一些有用的收集器，让我们先来看看。本章示例代码中的收集器都是从 `java.util.stream.Collectors` 类中静态导入的。

### 5.3.1 转换成其他集合
有一些收集器可以生成其他集合。比如前面已经见过的 `toList`，生成了 `java.util.List` 类 的实例。还有 `toSet` 和 `toCollection`，分别生成 `Set` 和 `Collection` 类的实例。到目前为止， 我已经讲了很多流上的链式操作，但总有一些时候，需要最终生成一个集合——比如：
- 已有代码是为集合编写的，因此需要将流转换成集合传入;
- 在集合上进行一系列链式操作后，最终希望生成一个值;
- 写单元测试时，需要对某个具体的集合做断言。

通常情况下，创建集合时需要调用适当的构造函数指明集合的具体类型:

```java
    List<Artist> artists = new ArrayList<>();
```
但是调用 `toList` 或者 `toSet` 方法时，不需要指定具体的类型。`Stream` 类库在背后自动为你挑选出了合适的类型。本书后面会讲述如何使用 `Stream` 类库并行处理数据，收集并行操作的结果需要的 `Set`，和对线程安全没有要求的 `Set` 类是完全不同的。

可能还会有这样的情况，你希望使用一个特定的集合收集值，而且你可以稍后指定该集合的类型。比如，你可能希望使用 `TreeSet`，而不是由框架在背后自动为你指定一种类型的 `Set`。此时就可以使用 `toCollection`，它接受一个函数作为参数，来创建集合如下例子所示：

- 使用 toCollection，用定制的集合收集元素

```java
    List<Integer> numbers =  Arrays.asList(1,2,3,4,1);
    TreeSet<Integer> treeSet = numbers.stream()
                                        .collect(Collectors.toCollection(TreeSet::new));
    assertEquals(treeSet.size(),numbers.size() - 1);
```

### 5.3.2 转换成值
还可以利用收集器让流生成一个值。`maxBy` 和 `minBy` 允许用户按某种特定的顺序生成一个值。如下代码展示了如何找出成员最多的乐队。它使用一个 `Lambda` 表达式，将艺术家映射为成员数量，然后定义了一个比较器，并将比较器传入 `maxBy` 收集器。

- 找出成员最多的乐队

```java
    public Optional<Artist> biggestGroup(Stream<Artist> artists){
        Function<Artist,Long> getCount = artist -> artist.getMembers().count();
        return artists.collect(Collectors.maxBy(comparing(getCount)));
    }

```
`minBy` 就如它的方法名，是用来找出最小值的。

还有些收集器实现了一些常用的数值运算。让我们通过一个计算专辑曲目平均数的例子来 看看

- 找出一组专辑上曲目的平均数

```java
    public double averageNumberOfTracks(List<Album> albums) { return albums.stream()
            .collect(averagingInt(album -> album.getTracks().size()));
    }
```

和以前一样，通过调用`stream`方法让集合生成流，然后调用`collect`方法收集结果。 `averagingInt` 方法接受一个 `Lambda` 表达式作参数，将流中的元素转换成一个整数，然后再计算 平均数。还有和 `double` 和 `long` 类型对应的重载方法，帮助程序员将元素转换成相应类型的值。

第 4 章介绍过一些特殊的流，如 `IntStream`，为数值运算定义了一些额外的方法。事实上， `Java 8`也提供了能完成类似功能的收集器，如`averagingInt`。可以使用`summingInt`及其重载方法求和。`SummaryStatistics` 也可以使用 `summingInt` 及其组合收集

### 5.3.3 数据分块
另外一个常用的流操作是将其分解成两个集合。假设有一个艺术家组成的流，你可能希望将其分成两个部分，一部分是独唱歌手，另一部分是由多人组成的乐队。可以使用两次过滤操作，分别过滤出上述两种艺术家。

但是这样操作起来有问题。首先，为了执行两次过滤操作，需要有两个流。其次，如果过 滤操作复杂，每个流上都要执行这样的操作，代码也会变得冗余。

幸好我们有这样一个收集器 `partitioningBy`，它接受一个流，并将其分成两部分它使用 `Predicate` 对象判断一个元素应该属于哪个部分，并根据布尔值返回一个`Map`到列表。因此，对于`true List`中的元素，`Predicat`e返回`true`;对其他`List`中的 元素，`Predicate` 返回 `false`。

使用它，我们就可以将乐队(有多个成员)和独唱歌手分开了。在本例中，分块函数指明艺术家是否为独唱歌手。

- 将艺术家组成的流分成乐队和独唱歌手两部分

```java
    public Map<Boolean, List<Artist>> bandsAndSolo(Stream<Artist> artists) {
        return artists.collect(partitioningBy(artist -> artist.isSolo()));
    }
```

- 也可以使用方法引用代替 Lambda 表达式

```java
    public Map<Boolean, List<Artist>> bandsAndSolo(Stream<Artist> artists) {
        return artists.collect(partitioningBy(Artist::isSolo);
    }
```

### 5.3.4 数据分组
数据分组是一种更自然的分割数据操作，与将数据分成 `ture` 和 `false` 两部分不同，可以使 用任意值对数据分组。比如现在有一个由专辑组成的流，可以按专辑当中的主唱对专辑分组。

- 唱对专辑分组

```java
    public Map<Artist, List<Album>> albumsByArtist(Stream<Album> albums) {
        return albums.collect(groupingBy(album -> album.getMainMusician()));
    }
```

和其他例子一样，调用流的 `collect` 方法，传入一个收集器。`groupingBy` 收集器(如图 5-2 所示)接受一个分类函数，用来对数据分组，就像 `partitioningBy` 一样，接受一个 `Predicate` 对象将数据分成 `ture` 和 `false` 两部分。我们使用的分类器是一个 `Function` 对 象，和 `map` 操作用到的一样。

__读者可能知道SQL中的group by操作，我们的方法是和这类似的一个概念，只不过在 Stream 类库中实现了而已。__

### 5.3.5 字符串
很多时候，收集流中的数据都是为了在最后生成一个字符串。假设我们想将参与制作一张专辑的所有艺术家的名字输出为一个格式化好的列表，以专辑  `Let It Be` 为例，期望的输出 为:"[George Harrison, John Lennon, Paul McCartney, Ringo Starr, The Beatles]"。

在 `Java 8` 还未发布前，实现该功能的代码可能如下代码所示。通过不断迭代列表，使用一 个  `StringBuilder` 对象来记录结果。每一步都取出一个艺术家的名字，追加到 `StringBuilder` 对象。

```java
    @Test
    public  void testAppend(){
        List<Artist> artists = new ArrayList<>();
        StringBuilder builder = new StringBuilder("[");
        for (Artist artist : artists) {
            if (builder.length() > 1) {
                builder.append(", ");
            }
            String name = artist.getName();
            builder.append(name);
        }
        builder.append("]");
        String result = builder.toString();
    }

```

显然，这段代码不是非常好。如果不一步步跟踪，很难看出这段代码是干什么的。使用 `Java 8` 提供的流和收集器就能写出更清晰的代码，如下代码所示。

```java
    @Test
    public  void testStreamJoin(){
        List<Artist> artists = new ArrayList<>();
        String result = artists.stream()
                .map(Artist::getName)
                .collect(Collectors.joining(",", "[", "]"));
        
    }
```

这里使用 `map` 操作提取出艺术家的姓名，然后使用 `Collectors.joining` 收集流中的值，该方法 可以方便地从一个流得到一个字符串，允许用户提供分隔符(用以分隔元素)、前缀和后缀。

### 5.3.6 组合收集器
虽然读者现在看到的各种收集器已经很强大了，但如果将它们组合起来，会变得更强大。

之前我们使用主唱将专辑分组，现在来考虑如何计算一个艺术家的专辑数量。一个简单的方案是使用前面的方法对专辑先分组后计数，如下代码所示。
- 计算每个艺术家专辑数的简单方式

```java
Map<Artist, List<Album>> albumsByArtist
         = albums.collect(groupingBy(album -> album.getMainMusician()));
Map<Artist, Integer> numberOfAlbums = new HashMap<>(); 
for(Entry<Artist, List<Album>> entry : albumsByArtist.entrySet()) {
         numberOfAlbums.put(entry.getKey(), entry.getValue().size());
     }         
```

这种方式看起来简单，但却有点杂乱无章。这段代码也是命令式的代码，不能自动适应并 行化操作。

这里实际上需要另外一个收集器，告诉 `groupingBy` 不用为每一个艺术家生成一个专辑列表，只需要对专辑计数就可以了。幸好，核心类库已经提供了一个这样的收集器: `counting`。使用它，可将上述代码重写为如下代码的样子。

```java
    public static Map<Artist,Long> numberOfAlbums(Stream<Album> albums){
        return albums.collect(groupingBy(album -> album.getMainMusician(),counting()));
    }
```

`groupingBy` 先将元素分成块，每块都与分类函数 `getMainMusician` 提供的键值相关联，然后使用下游的另一个收集器收集每块中的元素，最好将结果映射为一个 `Map`。

让我们再看一个例子，这次我们不想生成一组专辑，只希望得到专辑名。这个问题仍然可以用前面的方法解决，先将专辑分组，然后再调整生成的 Map 中的值

- 使用简单方式求每个艺术家的专辑名

```java
    public static Map<Artist, List<String>> nameOfAlbumsDumb(Stream<Album> albums) {
        Map<Artist, List<Album>> albumsByArtist =
                albums.collect(groupingBy(album -> album.getMainMusician()));
        Map<Artist, List<String>> nameOfAlbums = new HashMap<>();
        for (Map.Entry<Artist, List<Album>> entry : albumsByArtist.entrySet()) {
            nameOfAlbums.put(entry.getKey(), entry.getValue()
                    .stream()
                    .map(Album::getName).collect(toList()));
        }
        return nameOfAlbums;
    }
```

同理，我们可以再使用一个收集器，编写出更好、更快、更容易并行处理的代码。我们已经知道，可以使用 `groupingBy` 将专辑按主唱分组，但是其输出为一个 `Map<Artist, List<Album>>` 对象，它将每个艺术家和他的专辑列表关联起来，但这不是我们想要的，我们想要的是一个包含专辑名的字符串列表。

此时，我们真正想做的是将专辑列表映射为专辑名列表，这里不能直接使用流的 `map` 操 作，因为列表是由 `groupingBy` 生成的。我们需要有一种方法，可以告诉 `groupingBy` 将它 的值做映射，生成最终结果。

每个收集器都是生成最终值的一剂良方。这里需要两剂配方，一个传给另一个。谢天谢 地，`Oracle` 公司的研究员们已经考虑到这种情况，为我们提供了 `mapping` 收集器。

`mapping` 允许在收集器的容器上执行类似 `map` 的操作。但是需要指明使用什么样的集合类 存储结果，比如 `toList`。这些收集器就像乌龟叠罗汉，龟龟相驮以至无穷。

`mapping` 收集器和 `map` 方法一样，接受一个 `Function` 对象作为参数，经过重构后的如下代码所示。

```java
    public static Map<Artist, List<String>> nameOfAlbumsDumb1(Stream<Album> albums) {
        Map<Artist, List<String>> nameOfAlbums =
                albums.collect(groupingBy(Album::getMainMusician, mapping(Album::getName, toList())));
        return nameOfAlbums;
    }
```

这两个例子中我们都用到了第二个收集器，用以收集最终结果的一个子集。这些收集器叫作下游收集器。收集器是生成最终结果的一剂配方，下游收集器则是生成部分结果的配方，主收集器中会用到下游收集器。这种组合使用收集器的方式，使得它们在 `Stream` 类库 中的作用更加强大。

那些为基本类型特殊定制的函数，如 `averagingInt`、`summarizingLong` 等，事实上和调用特殊 `Stream` 上的方法是等价的，加上它们是为了将它们当作下游收集器来使用的。

### 5.3.7 重构和定制收集器
尽管在常用流操作里，`Java` 内置的收集器已经相当好用，但收集器框架本身是极其通用的。`JDK` 提供的收集器没有什么特别的，完全可以定制自己的收集器，而且定制起来相当简单，这就是本节要讲的内容。

读者可能还没忘记在之前的代码中，如何使用 `Java 7` 连接字符串，尽管形式并不优雅。让我们逐步重构这段代码，最终用合适的收集器实现原有代码功能。在工作中没有必要这样做， `JDK` 已经提供了一个完美的收集器 `joining`。这里只是为了展示如何定制收集器，以及如何使用 `Java 8` 提供的新功能来重构遗留代码。

- 使用 `for` 循环和 `StringBuilder` 格式化艺术家姓名

```java
    List<Artist> artists = new ArrayList<>();
    StringBuilder builder = new StringBuilder("[");
    for (Artist artist : artists) {
        if (builder.length() > 1) {
            builder.append(", ");
        }
        String name = artist.getName();
        builder.append(name);
    }
    builder.append("]");
    String result = builder.toString();
```

显然，可以使用 `map` 操作，将包含艺术家的流映射为包含艺术家姓名的流。如下代码展示了 使用了流的 `map` 操作重构后的代码。

```java
    List<Artist> artists = new ArrayList<>();
    StringBuilder builder = new StringBuilder("[");
    artists.stream().map(Artist::getName).forEach(name -> {
        if (builder.length() > 1)
            builder.append(", ");
        builder.append(name);
    });
    builder.append("]");
    String result = builder.toString();
```

将艺术家映射为姓名，就能更快看出最终是要生成什么，这样代码看起来更清楚一点。可惜 `forEach` 方法看起来还是有点笨重，这与我们通过组合高级操作让代码变得易读的目标不符。

暂且不必考虑定制一个收集器，让我们想想怎么通过流上已有的操作来解决该问题。和生 成字符串目标最近的操作就是 `reduce`，使用它将如下的代码重构如下。

- 使用 reduce 和 StringBuilder 格式化艺术家姓名

```java
    StringBuilder reduced =
            artists.stream()
                    .map(Artist::getName)
                    .reduce(new StringBuilder(), (builder1, name) -> {
                        if (builder1.length() > 0) {
                            builder1.append(", ");
                        }
                        builder1.append(name);
                        return builder1;
                    }, (left, right) -> left.append(right));
    reduced.insert(0, "[");
    reduced.append("]");
    String result = reduced.toString();
```
__这里比较疑惑的是第三个参数(left, right) -> left.append(right)，这里的left和right是指什么参数？__

上面👆的问题，经过调查[调查](https://www.cnblogs.com/h9527/p/5685439.html)，发现在普通 `stream` 中第三个 `lambda combiner`并不会执行，而在 `parallelStream`中会执行，如下解释：

>Executing this stream in parallel results in an entirely different execution behavior. Now the combiner is actually called. Since the accumulator is called in parallel, the combiner is needed to sum up the separate accumulated values.
>
>通过并行的方式执行上面的 stream 操作，得到的是另外一种完全不相同的执行动作。在并行 stream 中 combiner 方法会被调用。这是由于累加器是被并行调用的，因此组合器需要对分开的累加操作进行求和。

我曾经天真地以为上面的重构会让代码变得更清晰，可惜恰好相反，代码看起来比以 前更糟糕。让我们先来看看怎么回事。和前面的例子一样，都调用了 `stream` 和 `map` 方 法，`reduce` 操作生成艺术家姓名列表，艺术家与艺术家之间用“,”分隔。首先创建一 个 `StringBuilder` 对象，该对象是 `reduce` 操作的初始状态，然后使用 `Lambda` 表达式将姓名连接到 `builder` 上。`reduce` 操作的第三个参数也是一个 `Lambda` 表达式，接受两个 `StringBuilder` 对象做参数，将两者连接起来。最后添加前缀和后缀。

在接下来的重构中，我们还是使用 `reduc`e 操作，不过需要将杂乱无章的代码隐藏掉——我 的意思是使用一个 `StringCombiner` 类对细节进行抽象。代码如下所示。

- 使用 `reduce` 和 `StringCombiner` 类格式化艺术家姓名

```java
    @Test
    public  void testStringCombiner(){
        List<Artist> artists = Arrays.asList(new Artist("Zhangsan","China"),new Artist("Lisi","China"));
        StringCombiner combiner = artists.stream()
                .map(Artist::getName)
                .reduce(new StringCombiner(",","[","]"),
                        StringCombiner::add,
                        StringCombiner::merge);
        String result = combiner.toString();
    }
```

尽管代码看起来和上个例子大相径庭，其实背后做的工作是一样的。我们使用 `reduce` 操作将姓名和分隔符连接成一个 `StringBuilder` 对象。不过这次连接姓名操作被代理到了 `StringCombiner.add` 方法，而连接两个连接器操作被 `StringCombiner.merge` 方法代理。让我们现在来看看这些方法，先从下面中的 `add` 方法开始。

- `add` 方法返回连接新元素后的结果

```java
    public StringCombiner add(String element){
        if(areAtStart()){
            this.builder.append(this.prefix);
        } else {
            this.builder.append(this.delim);
        }
        builder.append(element);
        return this;
    }
```
`add` 方法在内部其实将操作代理给一个 `StringBuilder` 对象。如果刚开始进行连接，则在最 前面添加前缀，否则添加分隔符，然后再添加新的元素。这里返回一个 `StringCombiner` 对 象，因为这是传给 `reduce` 操作所需要的类型。合并代码也是同样的道理，内部将操作代理给 `StringBuilder` 对象，如下代码所示。

- `merge` 方法连接两个 `StringCombiner` 对象

```java
    public StringCombiner merge(StringCombiner other){
        this.builder.append(other.builder);
        return this;
    }
```

`reduce` 阶段的重构还差一小步就差不多结束了。我们要在最后调用 `toString` 方法，将整个步骤串成一个方法链。这很简单，只需要排列好 `reduce` 代码，准备好将其转换为 `Collector API` 就行了

- 使用 reduce 操作，将工作代理给 StringCombiner 对象

```java
    List<Artist> artists = Arrays.asList(new Artist("Zhangsan","China"),new Artist("Lisi","China"));
    String result = artists.stream()
                .map(Artist::getName)
                .reduce(new StringCombiner(",","[","]"),
                        StringCombiner::add,
                        StringCombiner::merge).toString();
```

__注意：实测发现使用 stream() 的确不会走 merge 方法，解释同上__

现在的代码看起来已经差不多完美了，但是在程序中还是不能重用。因此，我们想将 `reduce` 操作重构为一个收集器，在程序中的任何地方都能使用。不妨将这个收集器叫作 `StringCollector`，让我们重构代码使用这个新的收集器，如下代码所示。

- 使用定制的收集器 `StringCollector` 收集字符串

```java
    String result =
        artists.stream()
            .map(Artist::getName)
            .collect(new StringCollector(", ", "[", "]"));
```

既然已经将所有对字符串的连接操作代理给了定制的收集器，应用程序就不需要关心 `StringCollector` 对象的任何内部细节，它和框架中其他 `Collector` 对象用起来是一样的。

先来实现 `Collector` 接口如下代码，由于 `Collector` 接口支持泛型，因此先得确定一些具体的类型:
- 待收集元素的类型，这里是 `String`;
- 累加器的类型 `StringCombiner`;
- 最终结果的类型，这里依然是 `String`;

- 定义字符串收集器

```java
public class StringCollector implements Collector<String,StringCombiner,String> {}
```

一个收集器由四部分组成。首先是一个 `Supplier`，这是一个工厂方法，用来创建容器，在这个例子中，就是 `StringCombiner`。和 `reduce` 操作中的第一个参数类似，它是后续操作的初值，如下代码所示：

- `Supplier` 是创建容器的工厂

```java
    @Override
    public Supplier<StringCombiner> supplier() {
        return () -> new StringCombiner(delim, prefix, suffix);
    }
```

让我们一边阅读代码，一边看图，这样就能看清到底是怎么工作的。由于收集器可以并行收集，我们要展示的收集操作在两个容器上(比如 `StringCombiners` )并行进行。`StringCombiner` 中的 `merge` 方法就是为了处理流并行的情况。

收集器的每一个组件都是函数，因此我们使用箭头表示，流中的值用圆圈表示，最终生成的值用椭圆表示。收集操作一开始，`Supplier` 先创建出新的容器

![StringCollector](https://www.zhangaoo.com/upload/2018/11/51jk212sk6j38r2685rduf766v.png)

收集器的 `accumulator` 的作用和 `reduce` 操作的第二个参数一样，它结合之前操作的结果和当前值，生成并返回新的值。这一逻辑已经在 `StringCombiner` 的 `add` 方法中得以实现， 直接引用就好了

```java
    @Override
    public BiConsumer<StringCombiner, String> accumulator() {
        return StringCombiner::add;
    }
```

这里的 `accumulator` 用来将流中的值叠加入容器中如下图

![accumulator](https://www.zhangaoo.com/upload/2018/11/rg19rlag8giour2pb51q8munjq.png)

`combine` 方法很像 `reduce` 操作的第三个方法。如果有两个容器，我们需要将其合并。同样，在前面的重构中我们已经实现了该功能，直接使用 `StringCombiner.merge` 方法就行了

- combiner 合并两个容器

```java
    @Override
    public BinaryOperator<StringCombiner> combiner() { 
        return StringCombiner::merge;
    }
```

在收集阶段，容器被 `combiner` 方法成对合并进一个容器，直到最后只剩一个容器为止如下图

![alt](https://www.zhangaoo.com/upload/2018/11/2hmbm9gicqgrjo9b6e6jt9rcbn.png)

读者可能还记得，在使用收集器之前，重构的最后一步将 `toString` 方法内联到方法链的末端，这就将 `StringCombiner` 转换成了我们想要的字符串

![alt](https://www.zhangaoo.com/upload/2018/11/vobmj2tf32gg2on2v7oc870ng8.png)

收集器的 `finisher` 方法作用相同。我们已经将流中的值叠加入一个可变容器中，但这还不是我们想要的最终结果。这里调用了 `finisher` 方法，以便进行转换。在我们想创建字符串等不可变的值时特别有用，这里容器是可变的

- `finisher` 方法返回收集操作的最终结果

```java
    @Override
    public Function<StringCombiner, String> finisher() {
        return StringCombiner::toString;
    }
```

从最后剩下的容器中得到最终结果。

关于收集器，还有一点一直没有提及，那就是特征。特征是一组描述收集器的对象，框架
可以对其适当优化。`characteristics` 方法定义了特征，实现 `Collector` 接口，除了实现上面四个接口还需要实现 `Set<Characteristics> characteristics()` 接口。

需要注意上面实现的所有代码只起教学用途，对于拼接字符串的功能系统方法 `java.util.StringJoiner` 已经帮我们实现好了。

做这些练习的主要目的不仅在于展示定制收集器的工作原理，而且还在于帮助读者编写自 己的收集器。特别是你有自己特定领域内的类，希望从集合中构建一个操作，而标准的集 合类并没有提供这种操作时，就需要定制自己的收集器。

以 `StringCombiner` 为例，收集值的容器和我们想要创建的值(字符串)不一样。如果想要收集的是不可变对象，而不是可变对象，那么这种情况就非常普遍，否则收集操作的每一 步都需要创建一个新值。

想要收集的最终结果和容器一样是完全有可能的。事实上，如果收集的最终结果是集合， 比如 `toList` 收集器，就属于这种情况。

此时，`finisher` 方法不需要对容器做任何操作。更正式地说，此时的 `finisher` 方法其实是 `identity` 函数:它返回传入参数的值。如果这样，收集器就展现出 `IDENTITY_FINISH` 的特征，需要使用 `characteristics` 方法声明。

__小结：读到这里针对上面的问题: 这里的 left 和 right 是指什么参数？，因为流是可以并行处理的，因此在处理并行 accumulator 结果需要一个 merge 操作。上面的字符串收集器并不能完全处理并行流的情况，因为 merge 操作并未完全考虑并行情况下的 prefix、suffix Merge 操作，只是简单的 append 操作。__

### 5.3.8 对收集器的归一化处理
就像之前看到的那样，定制收集器其实不难，但如果你想为自己领域内的类定制一个收集器，不妨考虑一下其他替代方案。最容易想到的方案是构建若干个集合对象，作为参数传给领域内类的构造函数。如果领域内的类包含多种集合，这种方式又简单又适用。

当然，如果领域内的类没有这些集合，需要在已有数据上计算，那这种方法就不合适了。 但即使如此，也不见得需要定制一个收集器。你还可以使用 `reducing` 收集器，它为流上的归一操作提供了统一实现。如下代码展示了如何使用 `reducing` 收集器编写字符串处理程序。

这和上面的例子中讲到的基于 `reduce` 操作的实现很像，这点从方法名中就能看出。 区别在于 `Collectors.reducing` 的第二个参数，我们为流中每个元素创建了唯一的 `StringCombiner`。如果你被这种写法吓到了，或是感到恶心，你不是一个人!这种方式非常低效，这也是我要定制收集器的原因之一。

```java
    String  result =
            artists.stream()
            .map(Artist::getName)
            .collect(Collectors.reducing(
                    new StringCombiner(",","[","]"),
                    name -> new StringCombiner(", ", "[", "]").add(name),
                    StringCombiner::merge
            )).toString();
```

## 5.4 一些细节
`Lambda` 表达式的引入也推动了一些新方法被加入集合类。让我们来看看 `Map` 类的一些变化。

构建 `Map` 时，为给定值计算键值是常用的操作之一，一个经典的例子就是实现一个缓存。传统的处理方式是先试着从 `Map` 中取值，如果没有取到，创建一个新值并返回。

假设使用 `Map<String, Artist> artistCache` 定义缓存，我们需要使用费时的数据库操作查询艺术家信息，代码可能如下代码所示。

- 使用显式判断空值的方式缓存

```java
public Artist getArtist(String name) { 
    Artist artist = artistCache.get(name); 
    if (artist == null) {
        artist = readArtistFromDB(name);
        artistCache.put(name, artist);
    }
return artist; 
}
```

`Java 8` 引入了一个新方法 `computeIfAbsent`，该方法接受一个 `Lambda` 表达式，值不存在时使用该 `Lambda` 表达式计算新值。使用该方法，可将上述代码重写为如下所示的形式。

- 使用 `computeIfAbsent` 缓存
```java
public Artist getArtist(String name) {
    return artistCache.computeIfAbsent(name, this::readArtistFromDB);
}
```

你可能还希望在值不存在时不计算，为 `Map` 接口新增的 `compute` 和 `computeIfAbsent` 就能处理这些情况。

在工作中，你可能尝试过在 `Map` 上迭代。过去的做法是使用 `value` 方法返回一个值的集合， 然后在集合上迭代。这样的代码不易读。如下代码展示了本章早些时候介绍的一种方式，创建一个 `Map`，然后统计每个艺术家专辑的数量。

- 一种丑陋的迭代 `Map` 的方式

```java
Map<Artist, Integer> countOfAlbums = new HashMap<>(); 
for(Map.Entry<Artist, List<Album>> entry : albumsByArtist.entrySet()) {
    Artist artist = entry.getKey();
    List<Album> albums = entry.getValue();
    countOfAlbums.put(artist, albums.size());
}
```

谢天谢地，`Java 8` 为 `Map` 接口新增了一个 `forEach` 方法，该方法接受一个 `BiConsumer` 对象为参数(该对象接受两个参数，返回空)，通过内部迭代编写出易于阅读的代码，关于内 部迭代请参考 3.1 节。使用该方法重写后的代码如下代码所示。

- 使用内部迭代遍历 `Map` 里的值

```java
 Map<Artist, Integer> countOfAlbums = new HashMap<>();
     albumsByArtist.forEach((artist, albums) -> {
         countOfAlbums.put(artist, albums.size());
     });
```

## 5.5 要点回
- 方法引用是一种引用方法的轻量级语法，形如: `ClassName::methodName`。
- 收集器可用来计算流的最终值，是 `reduce` 方法的模拟。
- Java 8 提供了收集多种容器类型的方式，同时允许用户自定义收集器。(像前面使用的 `toList`、`toSet` 都是收集器)

## 5.6 练习
1. 方法引用
回顾第 3 章中的例子，使用方法引用改写以下方法:
a. 转换大写的 `map` 方法;

```java
    List<String> collected = Stream.of("a", "b", "hello")
            .map(String::toUpperCase)
            .collect(toList());
```

b. 使用 `reduce` 实现 `count` 方法;

```java
    public static Map<String, Long> countWords(Stream<String> names) {
        return names.collect(groupingBy(name -> name, counting()));
    }

```

c. 使用 `flatMap` 连接列表。

```java
 artists.stream()
        .flatMap(Artist::getMembers
        .reduce(0,(acc,members) -> member.count())
```

2. 收集器
a. 找出名字最长的艺术家，分别使用收集器和第 3 章介绍过的 reduce 高阶函数实现。然后对比二者的异同:哪一种方式写起来更简单，哪一种方式读起来更简单?以下面的参数为例，该方法的正确返回值为 "Stuart Sutcliffe":

```java
Stream<String> names = Stream.of("John Lennon", "Paul McCartney",
          "George Harrison", "Ringo Starr", "Pete Best", "Stuart Sutcliffe");
```

- 使用收集器

```java
        Function<String,Integer> getLength = artistName -> artistName.length();
        Optional<String> name1 =  names.collect(Collectors.maxBy(comparing(getLength))).orElseThrow(RuntimeException::new);
        assertEquals("Stuart Sutcliffe",name1.get());
```

- 使用 reduce 高阶函数

```java
        String name = names.reduce("", (acc, ele) -> ele.length() > acc.length() ? ele : acc);
        assertEquals("Stuart Sutcliffe",name);
```

- 分析：语义上收集器比较清晰，方法名称能准确表达预期的操作

b. 假设一个元素为单词的流，计算每个单词出现的次数。假设输入如下，则返回值为一 个形如  [John → 3, Paul → 2, George → 1] 的Map:

```java
Stream<String> names = Stream.of("John", "Paul", "George", "John",
                                        "Paul", "John");
```

```java
    Stream<String> names = Stream.of("John", "Paul", "George", "John","Paul", "John");
    Map<String ,Long> result = names.collect(groupingBy(name -> name,counting()));
```

c. 用一个定制的收集器实现 `Collectors.groupingBy` 方法，不需要提供一个下游收集器，只需实现一个最简单的即可。别看 JDK 的源码，这是作弊!提示:可从下面这行代
码开始:

```java
//以下是参考答案，想了一会实在没太明确思路
package com.insightfullogic.java8.answers.chapter5;

import java.util.*;
import java.util.function.BiConsumer;
import java.util.function.BinaryOperator;
import java.util.function.Function;
import java.util.function.Supplier;
import java.util.stream.Collector;

public class GroupingBy<T, K> implements Collector<T, Map<K, List<T>>, Map<K, List<T>>> {

    private final static Set<Characteristics> characteristics = new HashSet<>();
    static {
        characteristics.add(Characteristics.IDENTITY_FINISH);
    }

    private final Function<? super T, ? extends K> classifier;

    public GroupingBy(Function<? super T, ? extends K> classifier) {
        this.classifier = classifier;
    }

    @Override
    public Supplier<Map<K, List<T>>> supplier() {
        return HashMap::new;
    }

    @Override
    public BiConsumer<Map<K, List<T>>, T> accumulator() {
        return (map, element) -> {
            K key = classifier.apply(element);
            List<T> elements = map.computeIfAbsent(key, k -> new ArrayList<>());
            elements.add(element);
        };
    }
    @Override
    public BinaryOperator<Map<K, List<T>>> combiner() {
        return (left, right) -> {
            right.forEach((key, value) -> {
                left.merge(key, value, (leftValue, rightValue) -> {
                    leftValue.addAll(rightValue);
                    return leftValue;
                });
            });
            return left;
        };
    }
    @Override
    public Function<Map<K, List<T>>, Map<K, List<T>>> finisher() {
        return map -> map;
    }

    @Override
    public Set<Characteristics> characteristics() {
        return characteristics;
    }
}
```


3. 改进Map
使用 Map 的 computeIfAbsent 方法高效计算斐波那契数列。这里的“高效”是指避免将那些较小的序列重复计算多次。

```java
    public static Map<Integer,BigDecimal> fibCache = new HashMap<>();
    public static BigDecimal optimiseFib(Integer i){
        if (i == 1 || i== 2){
            return new BigDecimal(1);
        }
        Function<Integer,BigDecimal> compute = j -> {
            BigDecimal left = optimiseFib(j - 1);
            BigDecimal right = optimiseFib(j - 2);
            BigDecimal result =  left.add(right);
            fibCache.put(i,result);
            return result;
        };
        return fibCache.computeIfAbsent(i,compute);
    }

    @Test
    public  void testOptimiseFib(){
        System.out.println(optimiseFib(1500));
    }
```

- 斐波那契数列的而外探索
发现使用上面递归且加 `map` 缓存的方法，在计算第 `2000` 个斐波那契数的时候就报栈溢出了，如果不使用缓存发现求第 `50` 个斐波那契数都求不出来。
简单想了一下使用 `for` 循环来做，使用 `List` 来存储结果，代码如下：

```java
    public static BigDecimal listCalcFib(Integer i) {
        if (i < 3) {
            return new BigDecimal(1);
        }
        List<BigDecimal> result = Lists.newArrayListWithExpectedSize(i - 1);
        result.add(new BigDecimal(1));
        result.add(new BigDecimal(1));

        for (int j = 2; j < i; j++) {
            result.add(result.get(j - 2).add(result.get(j - 1)));
        }
        return result.get(i - 1);
    }
```
此时可以求出第 `25w` 个斐波那契数需要 `4327ms` 时间，再求第 `30w` 个的时候报了堆溢出，明显内存不够了 `java.lang.OutOfMemoryError: Java heap space`。

进一步想把 `List` 换成 `Array` 是不是能提升求解效率，代码如下：

```java
    public static BigDecimal arrayCalcFib(Integer i) {
        if (i < 3) {
            return new BigDecimal(1);
        }
        BigDecimal [] result = new BigDecimal[i];
        result[0] = new BigDecimal(1);
        result[1] = new BigDecimal(1);

        for (int j = 2; j < i; j++) {
            result[j] = result[j - 2].add(result[j - 1]);
        }
        return result[i - 1];
    }
```
发现当求解较小斐波那契数比如第 `1000` 个，`List`需要花费 `12ms`，而数组只要 `1ms`,但是发现当求很大，比如第 `20w` 个斐波那契数，第一次求解竟然`Array` 比 `List` 更慢。但是再求解一次，`List` 花费 `2822ms`，`Array` 花费 `2075ms`。猜想应该是大量申请内存导致内存扩张花费了不少时间。

仔细一想我并需要存储 `i-n` 之间的所有结果，只需要求解第 `n` 个，那是不是 3 个局部变量就能搞定呢？答案值肯定的，代码如下:

```java
    public static BigDecimal tmpVariableCalcFib(Integer i) {
        if (i < 3) {
            return new BigDecimal(1);
        }
        BigDecimal  result = new BigDecimal(0);
        BigDecimal first = new BigDecimal(1);
        BigDecimal second = new BigDecimal(1);

        for (int j = 2; j < i; j++) {
            result = first.add(second);
            first = second;
            second = result;
        }
        return result;
    }
```
结果是第 `100w` 个也可以求出，时间花了 `21653ms`。而求第 `20w` 只需要花 `1348ms`,相比 `Array` 的 `2075ms` 快了不少。到这个时候花费的大部分时间应该都是循环中的大数加法，如果不使用文件系统缓存的话，应该很难再大幅提高了，这个时候我不禁想换其他语言试试，比如 `golang`。

代码如下：

```golang
func calcFib(i int) *big.Int {
	if i < 3{
		return big.NewInt(1)
	}
	result,first,second := big.NewInt(0),big.NewInt(1),big.NewInt(1)
	for j := 2; j < i;j++{
		result = first.Add(first,second)
		first = second
		second = result
	}
	return result
}
```
结果只花了 `150.907815ms` 简直亮瞎狗眼有木有！！相比 `Java` 最快的成绩 `1348ms` 提高了将近一个数量级。再来算一下 `100w` 个斐波那契数，相比 `Java` 的 `21894ms`，`golang` 只花了 `4191ms` 也是一个数量级的差距。随手计算了一下第 `200w` 个斐波那契数 `golang` 花了 `17063ms`，大家猜第 `200w` 斐波那契数有多长，简单数了一下 `417975` 位长的数字。语言差距有如此之大吗？反正远远超出了我的预想。单单从这方面看，`golang` 确实有点牛逼呀！💯
