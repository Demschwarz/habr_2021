#Как я запускал классификацию изображений на домашнем кластере Apache Ignite ML

Я - студент университета, знаком с машинным обучением в рамках пройденного курса, есть
интерес к современным кластерным технологиям, конкретно Apache Ignite. Возникла идея
копнуть глубже, то есть немного разобраться с относительно не популярной библиотекой
[Apache Ignite Machine Learning](https://ignite.apache.org/docs/latest/machine-learning/machine-learning#machine-learning).
Под катом — история о том, как мне удалось запустить пример OneVsRestClassificationExample,
сначала пример сам по себе, а потом его же на датасете
[MNIST database of handwritten digits](https://www.openml.org/d/554).
Ну и еще немного про производительность кластера из одного и двух домашних ПК. Весь
код доступен в репозитории [HandwrittenSVM](??).

## История вопроса
Документация [Apache Ignite](https://ignite.apache.org/) описывает технологию как
"distributed supercomputer for low-latency calculations, .. and machine learning".
Захотелось это проверить — как кластер будет вести себя на относительно крупном датасете,
особенно если в кластере несколько узлов.

Изначально было интересно поработать с изображениями. Конкретно - с нейросетью, способной относить неизвестное
изображение к одному из заранее известных классов. Выбираю один из классических алгоритмов
классификации — алгоритм [SVM (Solving Vector Machine)](https://habr.com/ru/post/428503/).

## Начинаю работу
Изучаю документацию Apache Ignite ML, нахожу там раздел
[multiclass classification](https://ignite.apache.org/docs/latest/machine-learning/multiclass-classification#multiclass-classification),
там говорится про мою задачу на примере Glass dataset из 
[UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/datasets/Glass+Identification).
В репозитории Apache Ignite нахожу готовый код примера
[OneVsRestClassificationExample](https://github.com/apache/ignite/blob/master/examples/src/main/java/org/apache/ignite/examples/ml/multiclass/OneVsRestClassificationExample.java). 
Остается клонировать и запустить. 

Раздел [Getting Started](https://ignite.apache.org/docs/latest/machine-learning/machine-learning#getting-started)
обещает быстрый старт, ну я и пробую: клонирую, открываю, компилирую

Но что-то идет не так

![](https://habrastorage.org/webt/kv/po/qy/kvpoqyrz3dqumgh_sxvoanc1b6o.png)

С Maven и IntelliJ IDEA работаю недавно, как справляться с подобными ошибками, непонятно.

Начинаю гуглить в поисках решения или хотя бы товарищей по несчастью
— попадается на глаза статья
[Как быстро загрузить большую таблицу в Apache Ignite через Key-Value API](https://habr.com/ru/post/526708/).
Отличает её от большинства статей то, что в её тексте есть ссылка на маленький репозиторий с кодом. Пробую клонировать/открыть
в IntelliJ Idea/скомпилировать/выполнить, все запускается и работает как обещано.
Почему-то [автор](@vtch) использует библиотеки из
[GRIDGAIN Community Edition](https://www.gridgain.com/products/in-memory-computing-platform),
заменяю в файле pom.xml на то же самое из APACHE Ignite, ибо продолжающееся студенчество вырабатывает 
у меня устойчивую приязнь к Open Source-продуктам.
Опять работает, вот с этого можно стартовать.

Делаю по образу и подобию свой собственный пустой проект, копирую туда пример OneVsRestClassificationExample,
еще некоторые файлы из репозитория Apache Ignite и нужные зависимости в `pom.xml`. Оказалось,
пример использует крошечный датасет 1987 года GLASS_IDENTIFICATION размером всего в 116 строк.

Классификация выполняется быстро, в кластере с одним узлом на моем домашнем ПК занимает буквально пару секунд.

![](https://habrastorage.org/webt/io/gj/tr/iogjtr91ntp6r_4vknoebcrns00.png)
## Как я искал другой датасет
Хочется протестировать алгоритм на более крупном датасете. Встречаю статью
[Алексея Зиновьева](@zaleslaw)) под названием 
[Apache Ignite ML: origins and development](https://zaleslaw.medium.com/apache-ignite-ml-origins-and-development-d49a19e67202).
В статье есть отсылки к питоновскому пакету scikit-learn, поиск по этой теме приводит к туториалу
[Image Classification with MNIST Dataset](https://debuggercafe.com/image-classification-with-mnist-dataset).
Туториал рассказывает, как с помощью scikit-learn загрузить датасет mnist_784, нарисовать на экране загруженное, 
обучить сеть на наборе из 60 тысяч рисунков и протестировать качество обучения еще на 10 тысячах.

Смотрю дальше, оказывается, что на kaggle.com
[выложен](https://www.kaggle.com/oddrationale/mnist-in-csv?select=mnist_test.csv)
этот же датасет в удобном для обработки виде. Загружаю два CSV-файла: тестовый 
mnist_test.csv(17.46 MB, 10 тысяч рисунков) и обучающий mnist_train.csv(104.56 MB, 60 тысяч рисунков).
Первая колонка в файле — метка класса изображения, далее следует рисунок размером 28 х 28 (то есть строка
из 784 чисел от 0 до 255). Датасет содержит рукописные изображения арабских цифр, поэтому метка класса — число
от 0 до 9. 

## Как заменить датасет
Задача по замене оказалась несложной — в примере OneVsRestClassificationExample используется
enum [MLSandboxDatasets](ссылка_на_репозиторий), указывающий на расположение csv-файла в проекте. Там
даже есть параметр, указывающий на наличие заголовка в файле; первая колонка файла — метка класса.
Формат данных оказался тем же самым и добавление датасета свелось к добавлению строки

  ````
      MNIST_TRAIN_DATASET("sampleData/mnist_train.csv", true, ","),
  ````

## Запускаю и смотрю на время работы
Инструкция из
[Как быстро загрузить большую таблицу в Apache Ignite через Key-Value API](https://habr.com/ru/post/526708/),
говорит о том, как запустить кластер Apache Ignite в минимально достаточной конфигурации. В файле
`config\default-config.xml` выставляю в `true` параметр `peerClassLoadingEnabled`, указываю
ip-адрес, запускаю узел данных. В конфиге своего приложения `HandwrittenSVM.jar` включаю клиентский
режим `<property name="clientMode" value="true"/>` и тоже запускаю, все это в разных окнах одного
Windows-хоста. Узел данных запускается ОК, но как только начинает работать jar, вылетает ошибка

`Failed to find class with given class loader for unmarshalling (make sure same versions of all classes are available on all nodes or enable peer-class-loading) [clsLdr=sun.misc.Launcher$AppClassLoader@55f96302, cls=org.apache.ignite.ml.dataset.impl.cache.util.DatasetAffinityFunctionWrapper]
`

Как мне победить этот DatasetAffinityFunctionWrapper, если никаких собственных классов я не создавал и peerClassLoadingEnabled включал?

Спрашиваю в форуме Apache Ignite [в Telegram](https://t.me/RU_Ignite), оперативно получаю ответ
(спасибо за подсказку, @Thriftwy). Оказывается, чтобы запустить узел с поддержкой ML, надо скопировать
папку `optional\ignite-ml` в папку `libs`. Теперь узел при старте видит плагин

```
  [09:16:02] Configured plugins:
  [09:16:02]   ^-- ml-inference-plugin 1.0.0
  [09:16:02]   ^-- null
  [09:16:02]
```

Мой jar должен загрузить в кластер датасет `mnist_train.csv` из 60 тысяч изображений размером
(?) МБ. Смотрю в диспетчере
задач расход памяти узла данных во время того, как работает мой jar (верхняя строка) - впечатляет.

![](https://habrastorage.org/webt/ff/wa/oy/ffwaoy-msvcvte3wuqc4kabszcs.png)

В итоге всё вылетает с ошибкой, генерируемой самой системой (другого источника кириллицы здесь быть попросту не может).

![](https://habrastorage.org/webt/g6/s2/d_/g6s2d_nrz6dcun0aholnqwccld0.png)

Перевод на человеческий: `Удаленный хост принудительно разорвал существующее подключение`. (!)а на том
хосте в чем причина?

Приходится обходиться датасетом MNIST_TEST с 10 тысячами изображений, предназначенным для проверки
уже обученной сети. На нём алгоритм спокойно запускается — обучение сети в кластере из
одного узла данных и одного клиентского узла занимает 18 секунд. 

==

![](https://habrastorage.org/webt/ca/mq/ig/camqigggxbpmu7utkvfogur0q4u.png)

Понимаю, что со стандартной конфигурацией мало чего добьёшься. Начинаю искать способы увеличения выделяемой для 
узла данных памяти. В файле `ignite.sh` прописываю `JVM_OPTS="$JVM_OPTS -Xms512m -Xmx6000m"`, выделяя таким образом
6000 Мб памяти на узел. Предварительно уменьшив датасет до значений 20, 30 и 40 тысяч срок, меняя количество выделяемой
памяти, один за другим запускаю обучение. Получается запуститься на всех датасетах кроме полноразмерного `MNIST_TRAIN`,
выделенных 14 Гб всё же не хватает для завершения обучения.

Подключаю к локальной сети второй домашний компьютер, поменьше, на 8 Гб оперативной памяти, создавая тем самым вычислительный кластер. Провожу тесты на разных
датасетах, выделяя в сумме на двух машинах столько же памяти, сколько я выделял на этих же датасетах ранее, на одной машине.

Результаты, в целом, оказались ожидаемыми: 

![](https://habrastorage.org/webt/fo/gl/7h/fogl7hg7dchx4ansafngv5l5r3g.png)

Читаю код файла `OneVsRestClassificationExample.java`, становится понятно, что само обучение делится на четыре
последовательных части. Если я правильно всё понял, то первые два процесса отвечают за загрузку данных из датасета на узлы,
а остальные - за обучение сети.

Итого, принимая во внимание данные из таблицы, можем утверждать, что добавление второго узла данных положительно
влияет на время обучения сети.

