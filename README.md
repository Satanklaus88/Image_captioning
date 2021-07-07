# Image captioning

В этом репозитории находится финальный проект 2 семестра Deep Learning School. Для удобства проверки сохранена изначальная структура ноутбука с описаниями и комментариями преподавателя.

*Для проверяющих: каждая отдельная модель находится в своем ноутбуке: Basemodel.ipynb, AttentionBasemodel.ipynb, FlickRMobileNet.ipynb, FlickRInception.ipynb. Описание проведенной работы и выводы смотрите в readme*.

## Исходные данные

Для работы используется датасет [MSCOCO](https://cocodataset.org/#download) в форме, представленной преподавателем: уже векторизованные через InceptionV3 изображения размера [1,2048] и набор описаний к ним (по 5 описаний на изображение). При подготовке данных случайно выбиралось одно из 5 описаний для обучения.

В дополнительной части также был использован датасет [Flickr30k](https://www.kaggle.com/hsankesara/flickr-image-dataset): чуть более 30 тыс. изображений с 5 описаниями на каждое.

## Базовая модель

Для базовой модели ввиду того, что данные изображений были уже пропущены через энкодер, использовалась только RNN-архитектура на основе LSTM. В инференсе для использования на любых изображениях использовалась сперва изображение пропускалось через обученную InceptionV3.

Основная идея модели: CNN кодирует изображение вектором(матрицей), RNN инициализирует свои параметры из этого вектора и генерирует описание изображения:

При обучении сети были проведены попытки изменять TeacherForcing (вероятность при генерации получать на вход либо предыдущее сгенерированное слово, либо слово из исходного описания), но это не привело к каким-то конкретным результатам и в финальном варианте этот параметр был зафиксирован равным 1 (100% использование описания в генерации при обучении).

![](\pics\base_model.png)

## Базовая модель с Attention

К базовой модели был добавлен механизм Attention; был использован Bahdanau(Additive) Attention для предоставленных исходных данных. Полученный взвешенный вектор изображения конкатенировался с эмбеддингами слова при подаче в LSTM.

![](\pics\att20epochs.png)

## Датасет FlickR30K

С данным датасетом была опробована как энкодер модель MobileNetV2, в которой использовался и обученный линейный слой размерности 1000, и обучаемый в процессе на 2048. Хороших результатов добиться не удалось, модель недообучилась, судя по генерируемым ею описаниям. 

![mobilenetflickr](\pics\mobilenetflickr.png)

**Возможные дальнейшие улучшения:**

- Просто доучить модель в исходном варианте;
- Взять другие выходы из энкодера (не последний fc слой, а например свертки);
- Брать только определенные caption из датасета, а не случайные для каждой картинки (заметно, что стилистика и "сложность" описаний сильно различаются)
- Попробовать менять teacher forcing в процессе обучения
- Поискать ошибки в коде :)

Также попробовал взять предложенный Inception как энкодер; при обучении решил (на основе опыта выполнения домашних работ по курсу) не обращать внимание на растущий лосс на валидации и учить сеть дальше, ориентируясь лишь на лосс на трейне.

![mobilenetflickr](\pics\flickrinception.png)

Несмотря на такие графики, модель писала достаточно осмысленные описания (иногда :) (см. ниже).

## Полученные результаты

Получение описаний в проекте делалось 2-мя способами: слово за словом выбиралось либо самое вероятное("жадный" вариант), либо из распределения на основе вероятностей (сэмплирование).

#### Сгенерированные описания:

<img src="\pics\1.jpg" style="zoom:50%;" /> 

**"Жадный" вариант:** 

A man riding a skateboard down a ramp

**Сэмплирование:**  

A person is riding a skateboard on the ground

There is a man on a skateboard jumping on the air as another person extending his skateboard 

Players jumping on a ramp of practicing a scenery

 A man jumps high over a pile of rocks 

A skateboarder performing tricks on a skateboard close to legs

<img src="\pics\2.jpg" style="zoom:50%;" /> 

**"Жадный" вариант:** 

A cat is sitting on a couch next to a stuffed animal

**Сэмплирование:**  

A cat that is holding a yellow and white cat 

A cat is wearing a hat and blue tie 

A cat is sitting in a chair wearing a tie 

A cat with a hat is standing next to a stuffed animal 

A cat laying on a couch with a remote control

 <img src="\pics\3.jpg" style="zoom:100%;" />

**"Жадный" вариант:** 

A baby is eating a piece of pizza

**Сэмплирование:**  

A small child is holding a small white frisbee 

A baby is eating a piece of cake 

A small child is taking a picture of a fence 

A baby in a white shirt eating a carrot 

A black and white photo of a person with a toothbrush

<img src="\pics\4.jpg" style="zoom:100%;" /> 

**"Жадный" вариант:** 

A man in a suit and tie

**Сэмплирование:**  

A man in a suit and tie 

A man wearing a suit and tie 

A man wearing a jacket and tie 

A man wearing a suit and tie 

A man in a suit and tie

#### Выводы и дальнейшее развитие моделей

1. Опробовать различные виды сетей (ResNet, VGG)
2. Опробовать различные виды сетей Attention.
3. Попробовать изменять гиперпараметры существующих моделей, например увеличить размер словаря, изменять teacher forcing в обучении и т.д.
4. Попробовать другой оптимизатор (SGD, RMSProp) и увеличить при этом размер батча при обучении.
5. Реализовать beam search.

## Полезные ссылки на данную тему:

1. https://towardsdatascience.com/image-captioning-in-deep-learning-9cd23fb4d8d2

2. https://www.tensorflow.org/tutorials/text/image_captioning

3. https://www.hindawi.com/journals/cin/2020/3062706/

4. https://github.com/sgrvinod/a-PyTorch-Tutorial-to-Image-Captioning

5. https://cyberleninka.ru/article/n/zadachi-i-metody-avtomaticheskogo-opisaniya-izobrazheniy/viewer

6. https://github.com/yashk2810/Image-Captioning

   
