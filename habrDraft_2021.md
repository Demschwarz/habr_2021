#Как я запускал классификацию изображений на домашнем кластере Apache Ignite ML

Я - студент университета, знаком с машинным обучением в рамках пройденного курса, есть
интерес к современным кластерным технологиям, конкретно Apache Ignite. Возникла идея
копнуть глубже, то есть немного разобраться с относительно не популярной (во всяком 
случае, в русскоязычном сегменте) библиотекой
[Apache Ignite Machine Learning](https://ignite.apache.org/docs/latest/machine-learning/machine-learning#machine-learning).
Под катом — история о том, как мне удалось запустить пример OneVsRestClassificationExample,
сначала пример сам по себе, а потом его же на датасете
[MNIST database of handwritten digits](https://www.openml.org/d/554).
Ну и еще немного про производительность кластера из одного и двух домашних ПК. Весь
получившийся код доступен в репозитории [ссылка]

## История вопроса
Документация [Apache Ignite](https://ignite.apache.org/) описывает технологию как
"distributed supercomputer for low-latency calculations, .. and machine learning".
Захотелось проверить, как это все будет вести себя на датасете сколько-нибудь заметного размера, особенно
для кластера из нескольких узлов.

Изначально было интересно поработать с изображениями. Конкретно - с нейросетью, способной относить неизвестное
изображение к одному из заранее известных классов. Оказалось для таких задач принято использовать алгоритм
[SVM](https://habr.com/ru/post/428503/) ((!)Илья, прочитай комменты -- это про неосторожное высказывание
в публичной сети).

## Начинаю работу
Изучаю документацию, нахожу там раздел
[multiclass classification](https://ignite.apache.org/docs/latest/machine-learning/multiclass-classification#multiclass-classification),
там говорится про мою задачу на примере Glass dataset from the
[UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/datasets/Glass+Identification).
В репозитории Apache Ignite нахожу готовый код примера
[OneVsRestClassificationExample](https://github.com/apache/ignite/blob/master/examples/src/main/java/org/apache/ignite/examples/ml/multiclass/OneVsRestClassificationExample.java). 
Остается клонировать и запустить. 

Раздел [Getting Started](https://ignite.apache.org/docs/latest/machine-learning/machine-learning#getting-started)
обещает быстрый старт, ну я и пробую: клонирую, открываю ??, компилирую ??

Но что-то идет не так
![](https://habrastorage.org/webt/kv/po/qy/kvpoqyrz3dqumgh_sxvoanc1b6o.png) (скриншот)

В документации явно не хватает пошаговой инструкции по запуску.

Начинаю гуглить в поисках решения или хотя бы товарищей по несчастью
— попадается на глаза статья
[Как быстро загрузить большую таблицу в Apache Ignite через Key-Value API](https://habr.com/ru/post/526708/).
В тексте все круто (как всегда), но при этом есть маленький репозиторий с кодом. Пробую клонировать/открыть
в IntelliJ Idea/скомпилировать/выполнить, все запускается и работает как обещано.
Почему-то [автор](@vtch) использует библиотеки из
[GRIDGAIN Community Edition](https://www.gridgain.com/products/in-memory-computing-platform),
заменяю на то же самое из APACHE Ignite (студенты любят все бесплатное),
опять работает. Вот с этого можно стартовать.

Делаю по образу и подобию свой собственный пустой проект, кладу туда пример OneVsRestClassificationExample.
Положил в проект нужные файлы из репозитория Apache Ignite, добавил нужные зависимости в pom.xml. Оказалось,
пример использует крошечный датасет 1987 года GLASS_IDENTIFICATION размером всего в 116 строк.

(!)уточнить Пример обучает сеть и на том же датасете вычисляет матрицу погрешностей.

Классификация выполняется быстро, в кластере с одним узлом на моем домашнем ПК занимает == секунд

	<скрин>

## Другой датасет
В поисках более крупного датасета с изображениями встретил интересную статью 
[Apache Ignite ML: origins and development](https://zaleslaw.medium.com/apache-ignite-ml-origins-and-development-d49a19e67202),
оказалось, что реализация SVM Apache Ignite ML выполнена Алексеем Зиновьевым ([zaleslaw](??)).
В статье есть отсылки к питоновскому пакету scikit-learn, а там быстро нашелся туториал
[Image Classification with MNIST Dataset](https://debuggercafe.com/image-classification-with-mnist-dataset).
В нем идет речь о том, как с помощью scikit-learn обучить сеть на наборе из 70 тысяч графических образов
размером 28х28 точек, каждая точка принимает значение от 0 до 255. Датасет содержит 10 тысяч образов для
тестирования обченной сети.

== добавить как я загрузил датасет

## Запускаю
Если запустить два узла кластера на одном ПК по инструкции из [3], время расчета почти не меняется

    <скрин>

Добавил в кластер еще один ПК из домашней сети, время расчета изменилось

    <скрин>
	