![scrot](https://github.com/tvorozid/recommendation_system/blob/master/scrot.png)

[![GoTo](http://goto.msk.ru/templates/gk_university/images/favicon.ico)](http://goto.msk.ru/)
[![Code Health](https://landscape.io/github/xenx/recommendation_system/master/landscape.svg?style=flat)](https://landscape.io/github/xenx/recommendation_system/master)

## Предисловие
Находясь в лагере [GoTo](http://goto.msk.ru/) мы получили задачу рекомендательной системы от компании, которая в свою очередь получила заказ от телеканала '[Дождь](https://tvrain.ru/)'. Как только мы получили данные - мы начали работать.

Перед нами стояда задача проанализировать новостной портал и научиться автоматически рекомендовать пользователям новости, которые с большей вероятностью им понравятся.
Предложенная задача - реальная задача с которой столкнулись специалисты по анализу данных компании [E-contenta](https://e-contenta.com/ru) при разработке рекомендательного движка для сайта телеканала '[Дождь](https://tvrain.ru/)'. Часть данных была анонимизирована, как обычно делают компании, отдавая свои данные внешним аналитикам.

## Данные
Нам предоставили логи с сайта, которые содержат: время новости, ссылку на новость, id человека. Помимо этого нам дали текст новостей, заголовки (первый и второй), время новости, которое потом оказалось временем парсинга.

## Как мы это делали

### Метрика
Всё началось с выбора метрики, потому что есть очень много возможностей прострелить себе ноги оптимизируя не ту метрику. Для того, чтобы выбрать хорошую метрику, мы использовали [эту](https://habrahabr.ru/company/econtenta/blog/303458/) статью. В итоге мы выбрали MAP@K.

![MAP@K](https://tex.s2cms.ru/svg/map%40K%20%3D%20%5Cfrac%7B1%7D%7BN%7D%5Csum_%7Bj%3D1%7D%5EN%20ap%40K_j.) 

Важно заметить, что в метрике K = 10 ~ количество новостей, которое нужно рекомендовать.

### Topic Modeling
Мы начали с подбора алгоритма Topic Modeling-a. После долгих споров в команде и нескольких проверок и сравнений мы выбрали NMF - положительное разложение матрицы, которое по принцыпу похож на LDA, но при сравнении NMF - показал лучший результат.
О том, как использовать NMF в python можно прочитать [тут](https://de.dariah.eu/tatom/topic_model_python.html)

Для того, чтобы загнать текст в NMF модель его сначала нужно векторизовать. Мы использовали  [tf–idf](https://en.wikipedia.org/wiki/Tf%E2%80%93idf) текст туда поступал из мешка слов. (можно использовать n-граммы - тогда результат, возможно, будет лучше).

В итоге с помощью Topic Modeling-a мы построили матрицу в которой была вероятность отношения всех новостей к какой-либо теме из 40.

### Бинарный классификатор
С помощью логов мы так же построили [график](https://github.com/tvorozid/recommendation_system/blob/master/scripts/graph_by_url.ipynb) просмотров от времени с начала выхода новости, по которому можно увидеть, что популярность новости падает после пяти часов. Таким образом, мы поняли, что пользователю, в основном, не инетересны новости срок жизни которых превышает 5 часов. 

По данным логов мы построили сеанс человека - 4 новости, который он читал подряд по времени. С помощью бинарного классификатора мы предсказываем на сколько четвёртая новость подходит первым трём. 

После того, как мы попробовали разные алгоритмы  для бинарной классификации мы выбрали [Random Forest Classifier](http://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html) поскольку он показал лучший результат из всех. В качестве фичей для бинарногго классификатора мы использовали 160 фичей Topic Modeling-a (по 40 на каждую новость) и 3 коэффициента Отиаи для 1, 2, 3 новости с 4. Этот коэффициент понять какие пользователи читают одни и те же новости.

## Первый результат готов, что дальше
После того, как мы написали 40 штук IPython тетрадок, с разными результатами мы решили переписать всё в нормальный вид и сделать демо версию, где пользователь будет выбирать 3 новости, которые он прочитал, а мы ему будем возвращать рекомендации. И в итоге у нас это получилось! Весь переписанный код находится в этои репозитории. Данные логов распространять нам не разрешили, поэтому сдесь их нет.

Спасибо организаторам [GoTo](http://goto.msk.ru/) и компании [E-contenta](https://e-contenta.com/ru/) за предоставленную задачу.
