## Процесс решения

Читаю описание задачи. Предоставлен набор из 2156 текстов новостных статей по различной тематике, собранных из интернета. Данные представлены в файле data.csv в виде текст-тема. Необходимо проанализировать данные тексты и построить модель, которая будет предсказывать тему новости. Звучит как постановка ML-задачи мультиклассовой классификации. Но надо смотреть в данные. Так что формулирую интересующие меня вопросы к данным, объясняю почему эти вопросы важные (на что повлияют ответы), и в ходе EDA раазведочного анализа данных отвечаю на них.


**Q1.** Много ли вообще данных? Чтобы понять какие модели можно применять, и не будет ли с фичами проблемы (табличка фичей более широкая чем длинная).

**A1.** Данных немного, хотелось бы хотя бы по тысяче примеров на класс. Поэтому буду тестировать простые модели. Еще можно было бы снижать размерность (PCA).


**Q2.** Сбалансированная ли выборка. Чтобы понять как считать метрики (мульти или макро усреднение), чтобы понять как делать валидацию.

**A2.** Несбалансированная. Топ3 темы занимают 70% датасета, а "не по теме" топ1. Есть совсем плохо представленные классы (по 4, 9, 10, 12 сэмплов на класс). Поэтому буду делать разделение трейна от теста с `stratify`. А еще ставить `class_weight=balanced` в модели, и смотреть метрику с микро-среднением. Решаю брать микро f1, так как гармоническое среднее (учитывает и то и то) между пресижном и покрытием, и микро лучше используется для дизбалансных данных.


**Q3.** Какие лейблы (темы), могут ли пересекаться. Это важно для мультикласс/мультилейбл постановки.

**A3.** 13 классов. Иерархии и пересечений буду считать что нет, хотя темы "Международные отношения", "Россия", "мнения" наверняка пересекаться могут по сути). Сразу проблема что класс "не по теме" очень большой! Поэтому, возможно, сначала необходимо обучать модель на отделять спам от не спама.


**Q4.** Как устроены тексты. Это важно для того, чтобы если много супер-коротких или супер-длинных, надо принимать меры.

**A4.** В тексте всегда три строки: заголовок (полезно!), сам текст, ссылка (прикольно что в теории для части можно просто маппинг сделать в темы, ссылки часто вида "tass.ru/sport",  "tass.ru/politika",  "vedomosti.ru/politics", "mk.ru/culture", "mk.ru/politics", "mk.ru/economics", "vz.ru/opinions", "regnum.ru/news/society", "rosbalt.ru/russia"). Что касается длин текстов, то с разбивкой по категориям надо смотреть, там сильно отличаются, поэтому чтобы не зашумлять фичи буду брать только часть текста, обрезая.


## Схема решения

`текст новости -> title*2 + обрезанный текст -> POS-filter и нормализация -> усредненные эмбединги фасттекст модели -> модель бинарной классификации на спам/не спам -> модель мультиклассовой классификации по всем другим классам ->тема новости`

Возьму такую модель, которая легко позволяет мультикласс. А не OneVSRest, для быстроты.

## Что дальше

- модель можно взять специально обученную на новостях, word2vec с https://rusvectores.org/ru/
- в 2022 году, наверное, можно взять зиро-шот мультилейбл классификацию с хаггингфес: https://huggingface.co/MoritzLaurer/mDeBERTa-v3-base-mnli-xnli
- телеграм-бот. легче чем поднимать апи
- если много времени, то большие трансформенные (BERT) модели на открытых датасетах поучить (лента-ру вроде есть на гигабайты и много тем там), потом заменить классификационную голову трансформеру на наши классы)
- с муссорным классом можно было поступить иначе. Полностью убрать этот класс и примеры этого класса. А модель вызывать не в режиме predict, а в режиме predict_proba. То есть получать вероятности принадлежности к каждому из оставшихся классов. Подобрать порог, который если не достягаем, то приписываем мусорный класс.
