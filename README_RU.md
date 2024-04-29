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
  - `FastAPI`: - Используется для создания веб-приложения и обработки HTTP запросов, поддерживает асинхронные операции.
  - `aiohttp`: - Применяется для асинхронных HTTP запросов, например, для загрузки изображений по URL.
  - `OpenCV (cv2)`: - Библиотека компьютерного зрения, используется для работы с изображениями, включая их загрузку, изменение размера и сравнение.
  - `numpy`: - Библиотека для работы с многомерными массивами, используется в связке с OpenCV для обработки изображений.
  - `skimage`: - В частности, функция `structural_similarity` используется для сравнения схожести изображений.
  - `hashlib`: Используется для генерации хеша изображений, что позволяет уникально идентифицировать каждое изображение.

- **_Поддерживает множество индексов_** 🚀

  - `Индекс структурного сходства (SSIM)` ([подробности](https://en.wikipedia.org/wiki/Structural_similarity_index_measure))
  - `Сопоставление признаков с использованием дескриптора ORB (Oriented FAST and Rotated BRIEF)` ([подробности](https://en.wikipedia.org/wiki/Oriented_FAST_and_rotated_BRIEF))
  - `Изменение размера и преобразование в оттенки серого` ([подробности](https://en.wikipedia.org/wiki/Grayscale))
  - `Хеширование для идентификации изображений`
    
## ⚠️ Подготовка к работе: ⚠️

### _Для работы с Echo Image убедитесь, что у вас установлены следующие компоненты:_
- Python 3.8 или выше.
- pip (Python Package Installer):
1. **_Клонировать репозиторий:_** ✔️
- `git clone https://github.com/<Solrikk>/EchoImage.git`
- `cd EchoImage`
2. **_Настройка виртуальной среды:_** ✔️
```ShellScript
python -m venv venv
# Windows
venv\Scripts\activate
# Linux и MacOS
source venv/bin/activate
```
3. **_Устанавливать зависимости:_** ✔️
```ShellScript
pip install -r requirements.txt
```
### _Запуск приложения:_
1. **_Start the server:_**
```ShellScript
python app.py
```
`After starting the server, the application will be available at http://localhost:5000 .`

## Пример: 📋
(**_Код с коментариями_**)

```Python
# Определение асинхронной функции для обработки и сравнения изображения с целевым изображением.
async def process_image(session, image_entry, target_image):
  try:
    # Получение списка URL-адресов изображений с веб-страницы.
    image_urls = await get_image_urls_from_page(session, image_entry["url"])
    for image_url in image_urls:
      # Загрузка текущего изображение по URL-адресу.
      current_image = await download_image(session, image_url)
      # Определение оптимального размера для сравнения, не превышающий 1024 пикселей.
      optimal_size = max(max(target_image.shape[:2]),
                         max(current_image.shape[:2]))
      optimal_size = min(1024, optimal_size)
      # Изменение размера как целевого, так и текущего изображения до оптимального размера для сравнения.
      target_image_resized = cv2.resize(target_image,
                                        (optimal_size, optimal_size))
      current_image_resized = cv2.resize(current_image,
                                         (optimal_size, optimal_size))
      # Преобразование изображения в оттенки серого для процесса сравнения.
      target_gray = cv2.cvtColor(target_image_resized, cv2.COLOR_BGR2GRAY)
      current_gray = cv2.cvtColor(current_image_resized, cv2.COLOR_BGR2GRAY)
      # Рассчёт индекса структурного сходства (SSIM) между двумя изображениями.
      ssim_index = ssim(target_gray, current_gray)
      # Инициализирования детектора ORB для извлечения объектов.
      orb = cv2.ORB_create(nfeatures=500)
      # Detect keypoints and compute descriptors for both images.
      target_keypoints, target_descriptors = orb.detectAndCompute(
          target_gray, None)
      current_keypoints, current_descriptors = orb.detectAndCompute(
          current_gray, None)
      # Return early if no descriptors are found in either image.
      if target_descriptors is None or current_descriptors is None:
        return (0, image_entry["url"])
      # Setup parameters for FLANN based matcher, used for finding good matches.
      index_params = dict(algorithm=6,
                          table_number=6,
                          key_size=12,
                          multi_probe_level=1)
      search_params = dict(checks=50)
      flann = cv2.FlannBasedMatcher(index_params, search_params)
      # Match descriptors between the two images and filter good matches.
      matches = flann.knnMatch(target_descriptors, current_descriptors, k=2)
      good_matches = [m for m, n in matches if m.distance < 0.75 * n.distance]
      # Calculate the feature score based on good matches.
      feature_score = len(good_matches) / float(len(target_keypoints))
      # Compute histograms for both images in RGB channels.
      target_hist = cv2.calcHist([target_image_resized], [0, 1, 2], None,
                                 [32, 32, 32], [0, 256, 0, 256, 0, 256])
      current_hist = cv2.calcHist([current_image_resized], [0, 1, 2], None,
                                  [32, 32, 32], [0, 256, 0, 256, 0, 256])
      # Normalize histograms.
      cv2.normalize(target_hist, target_hist)
      cv2.normalize(current_hist, current_hist)
      # Compare histograms using correlation method.
      hist_score = cv2.compareHist(target_hist, current_hist,
                                   cv2.HISTCMP_CORREL)
      # Calculate the final score by averaging SSIM, feature, and histogram scores.
      final_score = (feature_score + ssim_index + hist_score) / 3
      return (final_score, image_entry["url"])
  except Exception as e:
    # Handle any errors during the process and return a zero score.
    print(f"Failed to process image {image_entry['url']} due to {e}")
    return (0, image_entry["url"])
```

-----------------

![image](https://wikimedia.org/api/rest_v1/media/math/render/svg/4203f29f732e5cdc9d8a95907ef6d8e12f08ca09)

SSIM сравнивает паттерны изменений интенсивности пикселей, которые являются важными атрибутами для человеческого зрения. Оценка SSIM находится в диапазоне от `-1 до +1`, где `значение 1` указывает на идентичные изображения. Процесс можно разбить на три компонента:

<img src="https://github.com/Solrikk/EchoImage/blob/main/assets/ssim/ssim2.png" width="95%" /> 

1) **_Сравнение яркости_** позволяет оценить общую яркость изображений. Яркость в SSIM измеряется как среднее всех значений пикселей.
```Python
target_gray = cv2.cvtColor(target_image_resized, cv2.COLOR_BGR2GRAY)
current_gray = cv2.cvtColor(current_image_resized, cv2.COLOR_BGR2GRAY)
ssim_index = ssim(target_gray, current_gray)
```
2) **_Сравнение контраста_** измеряется через дисперсию интенсивности пикселей (отклонения от среднего значения), позволяя понять, насколько похожи распределения света и тени между двумя изображениями. Гистограмма нормализуется перед сравнением, чтобы обеспечить адекватное сравнение контрастности.
```Python
cv2.normalize(target_hist, target_hist)
cv2.normalize(current_hist, current_hist)
hist_score = cv2.compareHist(target_hist, current_hist, cv2.HISTCMP_CORREL)
```
3) **_Сравнение структуры_** сравнивает узоры пространственного распределения пикселей, игнорируя изменения в яркости и контрасте. Это достигается путем расчета ковариации между изображениями относительно их локальных средних значений.
```Python
ssim_index = ssim(target_gray, current_gray)
```
![image](https://wikimedia.org/api/rest_v1/media/math/render/svg/96b4f1c3840c3707a93197798dcbfbfff24fa92b)
![image](https://wikimedia.org/api/rest_v1/media/math/render/svg/fcda97086476fa420b3b06568a0d202980a600d0)
![image](https://wikimedia.org/api/rest_v1/media/math/render/svg/1aebd62ba5b7e6ae47780ccfa659333f078d6eac)

Для сравнения изображений используется Индекс структурного сходства **(SSIM)** для оценки сходства между изображениями, а также алгоритм **ORB (Oriented FAST and Rotated BRIEF)** для обнаружения ключевых точек и их описаний.

## _ORB (Oriented FAST and Rotated BRIEF)_ 
ORB метод, используемый в компьютерном зрении, особенно популярный для задач, связанных с распознаванием объектов, сопоставлением изображений и отслеживанием. Этот метод сосредоточен на быстром поиске ключевых точек на изображениях и описании их таким образом, чтобы можно было эффективно сравнивать их. ORB обеспечивает быстрый и эффективный способ сопоставления изображений, несмотря на изменения в угле обзора, масштабе или освещении.

<img src="https://github.com/Solrikk/EchoImage/blob/main/assets/ORB/ORB3.png" width="95%" /> 

1) **Oriented FAST (Features from Accelerated Segment Test):** Этот компонент отвечает за обнаружение точек интереса (или ключевых точек) на изображении. Он быстро определяет углы или края, которые выделяются по сравнению с окружающими областями. Таким образом, можно идентифицировать значительные или уникальные секции изображения.

2) **Rotated BRIEF (Binary Robust Independent Elementary Features):** После нахождения ключевых точек необходимо создать описание каждой из них для возможности сравнения с ключевыми точками другого изображения. BRIEF генерирует краткое бинарное описание точек, но не устойчив к повороту изображений. Здесь на помощь приходит "повернутая" часть - ORB добавляет возможность стабильно описывать точки даже при поворотах изображений.

Сочетая эти два подхода, ORB обеспечивает быстрое и эффективное сопоставление изображений несмотря на изменения угла обзора, масштаба или освещения.

_Используя алгоритм ORB, определяются ключевые точки и дескрипторы как для текущего, так и для целевого изображений._

Найденные ключевые точки сравниваются друг с другом для определения совпадений. Эти совпадения позволяют оценить схожесть изображений с другой точки зрения, отличной от SSIM. Итоговый показатель схожести рассчитывается как среднее между показателем SSIM и относительным количеством совпадающих ключевых точек (используя алгоритм ORB), что обеспечивает комплексный подход к анализу схожести изображений.

В приложении EchoImage используются как методы SSIM, так и ORB для поиска изображений, похожих на загруженное. Вот упрощенное объяснение того, как каждый метод работает в контексте вашего приложения и способствует нахождению похожих изображений:

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
