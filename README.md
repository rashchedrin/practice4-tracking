# Практика 4. Детектирование особых точек и трекинг

## Цели

__Цель данной работы__ - получить общее представление о фундаментальных объектах
компьютерного зрения - особых точках. Рассмотреть одно из возможных применений -
трекинг объектов.

## Общая структура проекта

Данный репозиторий содержит:

  - `include` -- директория с заголовочным файлом, содержащим интерфейс трекера.
  - `samples` -- директория с исходным кодом приложения, которое запускает
    трекинг на видео. Имя алгоритма и путь к видео передаются параметрами
    командной строки. Кроме того, может быть указан файл с описанием
    ground-truth траектории, в этом случае приложение также выдаст метрики
    precision и recall для запускаемого трекера.
  - `dataset` -- директория с примерами видео. Для каждого видео имеется
    одноименный файл с расширением `.txt`, в котором указана ground-truth
    траектория.
  - `.gitignore` -- список файлов, находящихся в директории проекта,
     но игнорируемые git'ом.
  - `.travis.yml` -- конфигурационный файл для системы автоматического
     тестирования Travis-CI.
  - `CMakeLists.txt` -- общий файл для сборки проекта с помощью CMake.
  - `README.md` -- настоящий файл.

## Задачи

__Основные задачи__:

  1. Реализовать алгоритм трекинга Median Flow.
  2. Собрать качественные метрики на имеющемся наборе видео, дать им оценку.

__Дополнительные задачи__:

  1. Улучшить качественные оценки трекинга при помощи реализации вспомогательных
     процедур:
     - Использование различных способов фильтрации точек (фиксированный порог,
       среднее и т.п.).
     - Оценка масштаба.
     - Использование различных особых точек (Harris corners,
       `goodFeaturesToTrack`, SIFT/SURF/ORB features и т.п.).
     - Детектировать срыв трекинга.

## Общая последовательность действий

  1. Сделать форк upstream-репозитория, затем клонировать origin к себе на
     локальную машину. Для получения инструкций можно обратиться к разделу
     [Общие инструкции по работе с Git][git-intro]
     в [практической работе 1][practice1].
  2. Собрать проект с помощью CMake и MS VS (раздел
     [Сборка проекта с помощью CMake и MS VS][cmake-msvs]
     в [практической работе 1][practice1]). В результате успешной сборки в
     build-каталоге в директории `bin` должен появить исполняемый файл
     `tracking_sample.exe`.
  3. Запустить `tracking_sample.exe` для получения справки и разобраться с
     параметрами его запуска.
  4. Добавить новую реализацию интерфейса `Tracker`.
  5. Реализовать примитивную версию алгоритма Median Flow.
  6. Запустить реализованный трекер на различных видео из каталога `dataset`,
     проанализировать его качество.
  7. Реализовать полную версию алгоритма, проанализировать качество.
  8. Попытаться ещё улучшить качество (см. [Дополнительные задачи][tasks]).

## Детальная инструкция по выполнению работы

  1. Сделать форк upstream-репозитория, затем клонировать origin к себе на
     локальную машину. Для инструкций можно обратиться к разделу
     [Общие инструкции по работе с Git][git-intro]
     в [практической работе 1][practice1].
  2. Собрать проект с помощью CMake и MS VS (см. раздел
     [Сборка проекта с помощью CMake и MS VS][cmake-msvs]
     в [практической работе 1][practice1]). В результате успешной сборки
     в build-каталоге в директории `bin` долен появить исполняемый
     файл `tracking_sample.exe`.
  3. Запустить `tracking_sample.exe` для получения справки и разобраться с
     параметрами его запуска.
     1. Приложение можно запускать без параметров - тогда оно просто выдает справку.

     ```txt
     Tracker algorithm:
     Video name:
     Error: can't recognize tracking algorithm or open video
     Usage:
     tracking_sample.exe <tracker_algorithm> <video_name> <bounding_box or path_to_gt_file>

     Video examples can be found in "dataset" folder

     Bounding box should be given in format "x1,y1,x2,y2",
     where x's and y's are integer cordinates of opposed corners of bounding box

     Ground truth files are text files where each line is the representation
     of a bounding box in the described format. Examples can also be found
     in the"dataset" folder.

     Examples:

     $ ./bin/tracking_sample dummy ../dataset/car.mp4

     $ ./bin/tracking_sample dummy ../dataset/car.mp4 142,125,232,164

     $ ./bin/tracking_sample dummy ../dataset/car.mp4 ../dataset/car.txt
     ```

     2. Можно также передать 2 параметра так, как показано ниже. В
        результате откроется окно, в котором нужно будет мышью выбрать
        объект для трекинга. После чего будет показан результат трекинга
        с использованием указанного алгоритма (в данном случае - `dummy`).

     ```bash
     $ tracking_sample.exe dummy ../../dataset/car.mp4
     ```

     3. Можно передать 3 параметра так, как показано ниже. Тогда, кроме
        результатов указанного трекера, будет показана также разметка
        для сопровождаемого объекта (ground-truth). При этом по завершении
        видео, будут выведены качественные оценки алгоритма.

     ```bash
     $ tracking_sample.exe dummy ../../dataset/car.mp4 ../../dataset/car.txt
     ```

  4. Добавить новую реализацию интерфейса `Tracker`.
     1. Добавить новый файл с расширением `.cpp` в директорию `samples`
        (например, `tracker_median_flow_YOUR_NAME.cpp`).
     2. Перезапустить CMake, чтобы он "подхватил" созданный файл.
     3. Реализовать в созданном файле интерфейс `Tracker`, аналогично реализации
        в файле `tracker_dummy.cpp`. Тела методов могут быть пока тривиальными.

     ```cpp
     #include <tracker.hpp>

     class TrackerYourName : public Tracker
     {
     public:
         virtual ~TrackerYourName() {}

         virtual bool init( const cv::Mat& frame, const cv::Rect& initial_position );
         virtual bool track( const cv::Mat& frame, cv::Rect& new_position );

     private:
         cv::Rect position_;
     };

     bool TrackerYourName::init( const cv::Mat& frame, const cv::Rect& initial_position )
     {
         position_ = initial_position;
         return true;
     }

     bool TrackerYourName::track( const cv::Mat& frame, cv::Rect& new_position )
     {
         new_position = position_;
         return true;
     }
     ```

     4. В тот же файл после объявления класса необходимо поместить функцию создания
        собственного трекера

     ```cpp
     cv::Ptr<Tracker> createTrackerYourName()
     {
         return cv::Ptr<Tracker>(new TrackerYourName());
     }
     ```

     5. Добавить объвление своей функции `createTrackerYourName` в файл
        `trackers_factory.cpp`.

     ```cpp
     cv::Ptr<Tracker> createTrackerYourName();
     ```

     6. Вызов реализации полученной функции-фабрики необходимо поместить
        в функцию `createTracker` (файл `trackers_factory.cpp`), добавив
        дополнительную ветку в операторе условия.

     ```cpp
     if (impl_name == "dummy")
        return createTrackerDummy();
     else if (impl_name == "your_name"):
        return createTrackerYourName();
     ```

     7. Не забудьте исправить подсказку в функции `help` (файл
        `tracking_sample.cpp`), добавив информацию о возможности задания
        вашего трекера `your_name`.
  5. Реализовать примитивную версию алгоритма Median Flow. Метод `track` в
     созданной реализации должен содержать:
     1. Выбрать точки для сопровождения в прямоугольнике.
     2. Вычислить для них optical flow (следует использовать функцию 
        `calcOpticalFlowPyrLK`, [пример использования][lucas-canade-tutorial])
        и отфильтровать "плохие" точки. Фильтрация "плохих" точек
        осуществляется следующим образом:
        1. Отбросить точки, которые не протрекались, у них статус `false`.
        2. Выбрать медианную ошибку.
        3. Назначить "плохими" все точки, для которых ошибка больше медианной.
     3. Выбрать медианные смещения по X и по Y.
  6. Запустить реализованный трекер на различных видео из каталога `dataset`,
     проанализировать его качество. Попробуйте ответить на вопросы:
     1. Насколько полученные оценки близки к идеальным, равным 1? 
     2. В каких случаях алгоритм срабатывает плохо?
  7. Реализовать полную версию алгоритма, проанализировать качество. Шаги
     "полного" алгоритма следующие:
     1. Выбрать точки в прямоугольнике.
     2. Вычислить для них optical flow (и отбросить "плохие").
     3. *Вычислить обратный optical flow* (и отбросить "плохие" по forward-backward
        правилу). Фильтрация по forward-backward правилу осуществляется
        следующим образом:
        1. Вычислить optical flow в обратном направлении.
        2. Отфильтровать точки - если положение оригинальной точки и её образа,
           вычисленного с помощью forward-backward flow "сильно" отличаются, её нужно
           отбросить.
           1. Выбрать медианную ошибку.
           2. Назначить "плохими" все точки, для которых ошибка больше медианной.
     4. Взять медианные смещения по X и по Y.
     5. *Оценить масштаб*. Оценка масштаба осуществляется следующим образом:
        1. Для всех пар точек нужно определить отношение расстояния между ними на
           предыдущем и следующем кадре.
        2. Выбрать медианное отношение.
  8. Попытаться ещё улучшить качество (см. [Дополнительные задачи][tasks]).


<!-- LINKS -->

[practice1]: https://github.com/Itseez-NNSU-SummerSchool2015/practice1-devtools
[git-intro]: https://github.com/Itseez-NNSU-SummerSchool2015/practice1-devtools#Общие-инструкции-по-работе-с-git
[cmake-msvs]: https://github.com/Itseez-NNSU-SummerSchool2015/practice1-devtools#Сборка-проекта-с-помощью-cmake-и-microsoft-visual-studio
[tasks]: https://github.com/Itseez-NNSU-SummerSchool2015/practice4-tracking#Задачи
[lucas-canade-tutorial]: http://opencv-python-tutroals.readthedocs.org/en/latest/py_tutorials/py_video/py_lucas_kanade/py_lucas_kanade.html