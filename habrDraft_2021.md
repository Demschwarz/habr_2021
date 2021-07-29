#Как я запускал классификацию изображений на домашнем кластере Apache Ignite ML
Я - студент университета, знаком с машинным обучением в рамках пройденного курса, есть
интерес к современным кластерным технологиям, конкретно Apache Ignite. Возникла идея
копнуть глубже, то есть немного разобраться с
[Apache Ignite Machine Learning](https://ignite.apache.org/docs/latest/machine-learning/machine-learning#machine-learning).
Под катом — история о том, как мне удалось запустить пример OneVsRestClassificationExample,
сначала пример сам по себе, а потом его же на датасете
[MNIST database of handwritten digits](https://www.openml.org/d/554).
Ну еще немного про производительность кластера из одного и двух домашних ПК. Весь
получившийся код доступен в репозитории [ссылка]

## История вопроса
Документация [Apache Ignite](https://ignite.apache.org/) описывает технологию как
"distributed supercomputer for low-latency calculations, .. and machine learning".
Захотелось проверить, как будет вести себя пример на более крупном датасете, особенно
для кластера из нескольких узлов.

Раздел [Getting Started](https://ignite.apache.org/docs/latest/machine-learning/machine-learning#getting-started)
обещает быстрый страрт, ну я и пробую. Что-то идет не так

		<скрин>

Начинаю гуглить, ведь я не один такой неудачный, наверное есть товарищи по несчастью
— попадается на глаза статья
[Как быстро загрузить большую таблицу в Apache Ignite через Key-Value API](https://habr.com/ru/post/526708/).
В тексте все круто, как всегда, но при этом есть репозиторий с кодом.
То, что нужно для новичка!

Пробую клонировать/открыть в IntelliJ Idea/скомпилировать/выполнить, все работает.
Почему-то [автор](@vtch) использует библиотеки из
[GRIDGAIN Community Edition](https://www.gridgain.com/products/in-memory-computing-platform),
заменяю на то же самое из APACHE Ignite, опять работает. Вот с этого можно стартовать.

Положил в проект пример zaleslaw
[OneVsRestClassificationExample](https://github.com/apache/ignite/blob/master/examples/src/main/java/org/apache/ignite/examples/ml/multiclass/OneVsRestClassificationExample.java),
разобрался со ссылками. Оказалось, пример использует алгоритм [SVM](link!)
и выполняет классификацию образцов стекла по классическому датасету [GLASS_IDENTIFICATION].
Классификация выполняется быстро, в кластере с одним узлом на моем домашнем ПК занимает == секунд

	<скрин>

==


Если запустить два узла кластера на одном ПК по инструкции из [3], время расчета почти не меняется

    <скрин>

Добавил в кластер еще один ПК из домашней сети, время расчета изменилось

    <скрин>
	