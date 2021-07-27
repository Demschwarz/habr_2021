#Как я запускал классификацию изображений на домашнем кластере Apache Ignite ML

## История вопроса
попробовал запустить пример в лоб - не получилось

нашел статью
3 [Как быстро загрузить большую таблицу в Apache Ignite через Key-Value API](https://habr.com/ru/post/526708/), там про Ignite и есть репозиторий.

слушал курс по машинному обучению в универе, захотелось собрать своими руками

Взял структуру проекта из статьи [3], заменил содержимое на пример zaleslaw по адресу
[OneVsRestClassificationExample](https://github.com/apache/ignite/blob/master/examples/src/main/java/org/apache/ignite/examples/ml/multiclass/OneVsRestClassificationExample.java)
Пример [OneVsRestClassificationExample] использует алгоритм SVM и выполняет классификацию образцов стекла по классическому датасету [GLASS_IDENTIFICATION]. Классификация выполняется быстро, на моем домашнем ПК занимает == секунд

    <скрин>

Описание [Apache Ignite](https://ignite.apache.org/) описывает технологию как "distributed supercomputer for low-latency calculations, .. and machine learning". Захотелось проверить, как будет вести себя пример на более крупном датасете, особенно для кластера из нескольких узлов.

Взял датасет [mnist_784], время расчета примера выросло

    <скрин>

Если запустить два узла кластера на одном ПК по инструкции из [3], время расчета почти не меняется

    <скрин>

Добавил в кластер еще один ПК из домашней сети, время расчета изменилось

    <скрин>
	