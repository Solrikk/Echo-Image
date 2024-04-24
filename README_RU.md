<div align="center">
  <img src="assets/searching.png" width="30%"/>
</div>

<div align="center">
  <h3> <a href="https://github.com/Solrikk/Echo-Image/blob/main/README.md"> English | <a href="https://github.com/Solrikk/Echo-Image/blob/main/README_RU.md">Русский</a> | <a href="https://github.com/Solrikk/Echo-Image/blob/main/README_GE.md"> Deutsch </a> | <a href="https://github.com/Solrikk/Echo-Image/blob/main/README_JP.md"> 日本語 </a> | <a href="README_KR.md">한국어</a> | <a href="README_CN.md">中文</a> </h3>
</div>

-----------------

# Echo-Image ⚡️

 **_EchoImage:_** это передовая платформа, предназначенная для точного поиска похожих изображений. Используя алгоритмы сравнения структурного сходства и сопоставления ключевых точек, EchoImage обеспечивает быстрый и точный метод сравнения изображений. Приложение поддерживает загрузку изображений напрямую или через URL, эффективно просматривая огромную базу данных изображений для определения наилучших совпадений. Благодаря использованию асинхронных технологий, EchoImage обеспечивает быструю обработку, предлагая бесшовный и эффективный опыт визуального поиска.

**_Вы можете создать базу данных здесь_** - ([подробности](https://github.com/Solrikk/ImageSpaceDB))

## Возможности
- **_Поддерживает множество технологий_** ☄️

    _**Python**_ с библиотеками::
  - `FastAPI` для веб-фреймворка.
  - `aiohttp` для асинхронных HTTP-запросов.
  - `cv2` (OpenCV) для обработки изображений.
  - `numpy` для выполнения числовых операций.
  - `skimage` для дополнительных методов обработки изображений.

- **_Поддерживает множество индексов_** 🚀

  - `Индекс структурного сходства (SSIM)` ([подробности](https://en.wikipedia.org/wiki/Structural_similarity_index_measure))
  - `Сопоставление признаков с использованием дескриптора ORB (Oriented FAST and Rotated BRIEF)` ([подробности](https://en.wikipedia.org/wiki/Oriented_FAST_and_rotated_BRIEF))
  - `Изменение размера и преобразование в оттенки серого` ([подробности](https://en.wikipedia.org/wiki/Grayscale))
  - `Хеширование для идентификации изображений`
    
# Примеры
`Пример на Python` [[больше информации](https://github.com/Solrikk/EchoImage/blob/main/main.py)]

```Python
async def process_image(session, image_entry, target_image):
  try:
    current_image = await download_image(session, image_entry["url"])
    optimal_size = max(max(target_image.shape[:2]),
                       max(current_image.shape[:2]))
    optimal_size = min(1024, optimal_size)
    target_image_resized = cv2.resize(target_image,
                                      (optimal_size, optimal_size))
    current_image_resized = cv2.resize(current_image,
                                       (optimal_size, optimal_size))
    target_gray = cv2.cvtColor(target_image_resized, cv2.COLOR_BGR2GRAY)
    current_gray = cv2.cvtColor(current_image_resized, cv2.COLOR_BGR2GRAY)
    ssim_index = ssim(target_gray, current_gray)
    orb = cv2.ORB_create(nfeatures=500)
    target_keypoints, target_descriptors = orb.detectAndCompute(
        target_gray, None)
    current_keypoints, current_descriptors = orb.detectAndCompute(
        current_gray, None)
    if target_descriptors is None or current_descriptors is None:
      return (0, image_entry["url"])
    index_params = dict(algorithm=6,
                        table_number=6,
                        key_size=12,
                        multi_probe_level=1)
    search_params = dict(checks=50)
    flann = cv2.FlannBasedMatcher(index_params, search_params)
```

#

![image](https://wikimedia.org/api/rest_v1/media/math/render/svg/4203f29f732e5cdc9d8a95907ef6d8e12f08ca09)

SSIM сравнивает паттерны изменений интенсивности пикселей, которые являются важными атрибутами для человеческого зрения. Оценка SSIM находится в диапазоне от `-1 до +1`, где `значение 1` указывает на идентичные изображения. Процесс можно разбить на три компонента:

<img src="https://github.com/Solrikk/EchoImage/blob/main/assets/ssim/ssim2.png" width="95%" /> 

1) **_Сравнение яркости_** позволяет оценить общую яркость изображений. Яркость в SSIM измеряется как среднее всех значений пикселей.

```Python
target_gray = cv2.cvtColor(target_image_resized, cv2.COLOR_BGR2GRAY)
current_gray = cv2.cvtColor(current_image_resized, cv2.COLOR_BGR2GRAY)
ssim_index = ssim(target_gray, current_gray)
```

2) **_Сравнение контраста_** измеряется через дисперсию интенсивности пикселей (отклонения от среднего значения), понимая, насколько похожи паттерны распределения света и тени между двумя изображениями.
3) **_Сравнение структуры_** cравнивает паттерны пространственного распределения пикселей, игнорируя изменения в яркости и контрасте. Это делается путем расчета ковариации между изображениями относительно их локальных средних значений.

![image](https://wikimedia.org/api/rest_v1/media/math/render/svg/96b4f1c3840c3707a93197798dcbfbfff24fa92b)
![image](https://wikimedia.org/api/rest_v1/media/math/render/svg/fcda97086476fa420b3b06568a0d202980a600d0)
![image](https://wikimedia.org/api/rest_v1/media/math/render/svg/1aebd62ba5b7e6ae47780ccfa659333f078d6eac)

Для сравнения изображений используется Индекс структурного сходства **(SSIM)** для оценки сходства между изображениями, а также алгоритм **ORB (Oriented FAST and Rotated BRIEF)** для обнаружения ключевых точек и их описаний.

## _ORB (Oriented FAST and Rotated BRIEF)_ 
ORB метод, используемый в компьютерном зрении, особенно популярный для задач, связанных с распознаванием объектов, сопоставлением изображений и отслеживанием. Этот метод сосредоточен на быстром поиске ключевых точек на изображениях и описании их таким образом, чтобы можно было эффективно сравнивать их. ORB обеспечивает быстрый и эффективный способ сопоставления изображений, несмотря на изменения в угле обзора, масштабе или освещении.

<img src="https://github.com/Solrikk/EchoImage/blob/main/assets/ORB/ORB3.png" width="95%" /> 

1) **Oriented FAST (Features from Accelerated Segment Test):** This part is responsible for detecting points of interest (or key points) on the image. It quickly identifies corners or edges that stand out in comparison to their surrounding areas. This way, significant or unique sections of the image can be identified.

2) **Rotated BRIEF (Binary Robust Independent Elementary Features):** After key points have been found, it's necessary to create a description of each to allow comparison with key points from another image. BRIEF generates a brief binary description of the points but lacks resistance to image rotation. This is where the "rotated" part comes in - ORB adds the ability to stably describe points even when images are rotated.

By combining these two approaches, ORB provides a fast and efficient way of matching images despite changes in viewing angle, scale, or lighting.

_Using the ORB algorithm, key points and descriptors are determined for both the current and target images._

The found key points are compared with each other to determine matches. These matches allow assessing the similarity of images from a perspective other than SSIM. The final similarity score is calculated as the average between the SSIM score and the relative number of matching key points (using the ORB algorithm), providing a comprehensive approach to analyzing the similarity of images.

EchoImage application, both the SSIM and ORB methods are utilized to find images that are similar to an uploaded image. Here's a simplified explanation of how each method works in the context of your application and contributes to finding similar images:

## Как работает SSIM в EchoImage:
1) **_Масштабирование изображений:_** При сравнении загруженного изображения с каждым изображением в базе данных оба изображения масштабируются до одинаковых размеров (256x256 пикселей). Это стандартизирует сравнение, делая его справедливым и более эффективным, поскольку мы работаем с изображениями одинакового размера.

2) **_Конвертация в оттенки серого:_** Оба изображения конвертируются в оттенки серого. Это упрощает сравнение, фокусируясь на структуре и интенсивности света, а не отвлекаясь на различия в цвете.

3) **_Сравнение структурной схожести:_** Затем метод SSIM сравнивает эти изображения в оттенках серого для оценки их структурной схожести. Это включает анализ того, насколько похожи между двумя изображениями узоры света и тени, выдавая оценку, отражающую их схожесть. Высокая оценка означает структурную схожесть изображений.

## Как работает ORB в EchoImage:
1) **_Обнаружение ключевых точек:_** Сначала ORB определяет ключевые точки как в загруженном изображении, так и в каждом изображении базы данных. Эти ключевые точки - это различимые места, которые могут быть легко узнаны и сравнены между изображениями, такие как углы и интересные текстуры.

2) **_Описание ключевых точек:_** Для каждой обнаруженной ключевой точки ORB генерирует уникальный дескриптор, который резюмирует характеристики ключевой точки. Этот дескриптор делается инвариантным к повороту, что означает, что он описывает ключевую точку таким образом, который остается последовательным даже если изображение повернуто.

3) **_Сопоставление ключевых точек:_** Затем приложение сопоставляет ключевые точки между загруженным изображением и каждым изображением в базе данных, используя эти дескрипторы. Процесс сопоставления включает поиск ключевых точек в изображении базы данных, которые имеют дескрипторы, похожие на те, что в загруженном изображении.

4) **_Оценка совпадений:_** Чем больше ключевых точек совпадает между двумя изображениями, тем выше оценка схожести на основе ORB. Эта оценка отражает, сколько уникальных особенностей разделяют изображения.

## Комбинирование SSIM и ORB:
После расчета показателей схожести с использованием как SSIM, так и ORB для каждого сравнения изображений, Harmony-Image усредняет эти оценки для получения окончательного показателя схожести.
Изображения из базы данных затем ранжируются на основе их окончательных показателей схожести, и выбираются 5 наиболее похожих изображений.

## Финальный выбор похожих изображений:
Приложение фильтрует дубликаты URL, чтобы обеспечить разнообразный набор похожих изображений.
Оно возвращает URL самых похожих изображений, которые затем могут быть представлены пользователю.
По сути, ваше приложение использует комбинацию структурного анализа (SSIM) и сопоставления особенностей (ORB) для поиска и ранжирования изображений в вашей базе данных, которые наиболее похожи на изображение, загруженное пользователем. Этот двойной подход использует преимущества обоих методов, обеспечивая надежное и тонкое сравнение, которое выходит за рамки простого анализа пиксель за пикселем.
