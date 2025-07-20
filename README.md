# mle-template-case-sprint2

Добро пожаловать в репозиторий-шаблон Практикума для проекта 2 спринта. Ваша цель — улучшить ключевые метрики модели для предсказания стоимости квартир Яндекс Недвижимости.

Полное описание проекта хранится в уроке «Проект. Улучшение baseline-модели» на учебной платформе.

Здесь укажите имя вашего бакета: s3-student-mle-20250504-e374587ccc-freetrack
___
Для запуска проекта и установки библиотек необходимо выполнить следующие команды:  

```bash
git clone https://github.com/ozerge/mle-project-sprint-2-v001.git
cd mle-project-sprint-2-v001
pip install -r requirements.txt 
```

### Этап 1: Разворачивание MLflow и регистрация модели

#### 1. Для запуска mlflow выполнить команды:

```bash
cd mlflow_server
sh run_mlflow_server.sh
```
[Cкрипт для запуска сервера MLflow](mlflow_server/run_mlflow_server.sh)<br>

#### 2. Загрузка базовой модели и логирование модели

***Ноутбук загрузки модели, метрик и датасета проекта 1-го спринта***
```
mlflow_server/download_baseline_model.ipynb
```
[Загрузка базовой модели](mlflow_server/download_baseline_model.ipynb)<br>

***Ноутбук логирования***
```
mlflow_server/logging_baseline_model.ipynb
```
[Логирование загруженных данных](mlflow_server/logging_baseline_model.ipynb)<br>

перейти в MLflow UI по **http://127.0.0.1:5000/**:

***Конфигурация эксперимента MLflow***  
```
EXPERIMENT_NAME = "ozerge_PROJECT_SPRINT_2"
RUN_NAME = "logging_baseline_model"  
REGISTRY_MODEL_NAME = "baseline_model"
experiment_id = 31
run_id = "6c9b3f1eddaa4ec4ace99d1f56903966"
```
Модель, метрики, датасет проекта и ноутбук залогированы. 

### Этап 2: Исследовательский Анализ Данных (EDA)

***Ноутбук***
```
model_improvement/project_template_sprint_2.ipynb
```
[Основной ноутбук](model_improvement/project_template_sprint_2.ipynb)<br>

***Конфигурация эксперимента MLflow***  
```
EXPERIMENT_NAME = "ozerge_PROJECT_SPRINT_2"
RUN_NAME = "EDA"
experiment_id = 31
run_id = "aac9b6208500448e987482b3df63fa03"
```

Данные были предобработаны в проекте первого спринта, проанализированы и  отчищены от выбросов. Пропущенных значений нет.  

***Вывод:***  
- По ряду категориальных и бинарных признаков виден дисбаланс например:`building_type_int`, `is_apartment`, `has_elevator`  
- Все числовые признаки несмотря на удаление выбросов имеют хвосты.  
- С годами растет высота потолков, но уменьшается площадь помещений.    
- Корреляционный анализ показывает что наиболее сильные признаки это признаки которые непосредственно касаются размеров площадей:  
    - `total_area` (0.68) — самая сильная связь. Чем больше общая площадь, тем выше цена (логично).
    - `rooms` (0.55) — количество комнат также сильно влияет на цену.  
    - `living_area` (0.45) — жилая площадь коррелирует с ценой, но слабее, чем общая площадь.  

    - `total_area` и `rooms` (0.84) — почти линейная зависимость. Это может означать, что количество комнат часто определяется общей площадью (риск мультиколлинеарности).
    - `build_year` и `floors_total` (0.74) — в новых домах обычно больше этажей.
    - `kitchen_area` слабо связана с `price` (0.26), но сильно с `build_year` (0.39) — возможно, в новых домах делают большие кухни.
    - `ceiling_height` (0.33) — высота потолков имеет умеренное влияни.  

    - Географические координаты (`latitude, longitude`) почти не влияют на цену (коэффициенты близки к 0). Возможно, данные недостаточно детализированы (например, нет привязки к районам).
    - `flats_count` (количество квартир в доме) и `floors_total` слабо связаны с ценой — цена зависит скорее от характеристик самой квартиры, чем от дома.  

- Геолокации объектов сгруппированы в одной области и судя по данным это Москва. На будущее: можно сформировать доп. признаки, такие как удаленность от центра и от метро.  
- ***Таргет наиболее "приближен" к нормальному распределению, но наблюдается правосторонняя асимметрия. На базовой модели был логорифмирован целевой признак и наблюдалось резкое улучшение метрик.***  

- Вывод сохранен в директории [assets](model_improvement/assets/)<br>:  
    -`conclusions_eda.md`
- Все графики для анализа сохранены в отельные файлы директории [assets](model_improvement/assets/)<br>:  
    - `cat_features.png`  
    - `num_features.png`
    - `statistics_by_year.png`
    - `target.png`
    - `corr_with_target.png`
    - `boxplot_cat_features_price.png`
    - `geo_price.png`

Все графики, вывод и ноутбук залогированы.  

#### Этап 3: Генерация признаков и обучение модели  

***Ноутбук***  
```
model_improvement/project_template_sprint_2.ipynb  
```
[Основной ноутбук](model_improvement/project_template_sprint_2.ipynb)<br>  

***Конфигурация эксперимента MLflow***  
```
EXPERIMENT_NAME = "ozerge_PROJECT_SPRINT_2"
RUN_NAME = "feature_generation"
REGISTRY_MODEL_NAME = "project_2_model_generated_features"
experiment_id = 31
run_id = "1b0f055c8ed74f31898c3b344484b4c0"
```
[Выод](model_improvement/assets/feature_generation.md)<br>

***Вывод:***   
- Добавлены новые признаки: `ratio_living_area`, `ratio_kitchen_area`, `mean_flats_area`, `distance_to_center`, `total_rooms_ratio`. Для избежания мультиколлинеарности удален признак `rooms`.  
- Применение методов PolynomialFeatures и KBinsDiscretizer только ухудшило модель.  
- autofeat принес улучшение всех метрик, но на маленькие величины порядка 0.3 процента.

Модель, метрики и ноутбук залогированы.

#### Этап 4: Отбор признаков и обучение новой версии модели

***Ноутбук***  
```
model_improvement/project_template_sprint_2.ipynb  
```
[Основной ноутбук](model_improvement/project_template_sprint_2.ipynb)<br>  

***Конфигурация эксперимента MLflow***  
```
EXPERIMENT_NAME = "ozerge_PROJECT_SPRINT_2"
RUN_NAME = "feature_selection"
REGISTRY_MODEL_NAME = "project_2_model_feature_selection"
experiment_id = 31
run_id = "aff45617a5e547178d7fefe9299a7438"
```
- Для отбора признаков использовались два подхода из библиотеки mlxtend: SequentialFeatureSelector методом forward и backward.  
- Все графики для анализа сохранены в отельные файлы директории [assets_fs](model_improvement/assets_fs/)<br>:    
    - `sfs.png`  
    - `sbs.png`    
- Список выбранных признаков сохранен в директории [assets_fs](model_improvement/assets_fs/)<br>:
    - `selected_features_name.pkl`  

Модель, графики, метрики и ноутбук залогированы.  

#### Этап 5: Подбор гиперпараметров и обучение новой версии модели

***Ноутбук***  
```
model_improvement/project_template_sprint_2.ipynb  
```
[Основной ноутбук](model_improvement/project_template_sprint_2.ipynb)<br>  

***Конфигурация эксперимента MLflow***  
```
EXPERIMENT_NAME = "ozerge_PROJECT_SPRINT_2"
RUN_NAME = "hyper_params_serching"
REGISTRY_MODEL_NAME = "project_2_model_hyper_params_serching"
experiment_id:  31
run_id = "87b902d9d99243329868e4ffd9e1015d"
```
Для подбора гиперпараметров использовались методы Random Search и Bayesian Optimization через Optuna.
Random Search vs Bayesian Optimization  

Подбор гиперпараметров смог улучшить итоговое качество.    

- Random Search показал лучший результат (0.295827) по сравнению с Bayesian Optimization (0.331021).   
- Bayesian Optimization работал быстрее (8 минут против 13 минут). Random Search потребовал на ~60% больше времени.  
- Random Search показал лучшее качество модели, несмотря на большее время выполнения. Если время не критично - выбираем его. Если важно быстро получить хороший результат - Bayesian Optimization.  
- Оба метода нашли разные оптимальные параметры.  
- Random Search выбрал более высокую learning_rate (0.05 vs 0.001) и больше iterations (1500 vs 1205).  
- Выбираем Random Search, так как он дал лучший RMSE. Разница в качестве существенна (~11.9% улучшения).
- Итоговые параметры,вывод и таблица сравнения двух методов сохранены в директории [assets_hyp](model_improvement/assets_hyp/)<br>:
    - `best_params_final.json`
    - `summary.md`
    - `results.csv`

Модель, файлы, метрики и ноутбук залогированы.  

***Итоговый вывод:***  

- Все метрики улучшились.
- Дальнейшее улучшение возможно на стадии предобработки данных.
- Добавить новые признаки на основе расстояния от центра, которые поделят недвижимость на элитную и спальных районов. Разделить на категории:  
    - внутри Садового кольца
    - от Садового кольца до Третьего транспортного кольца
    - от ТТК до МКАД.
    - от МКАД - за МКАД
- Дальнейшее возможное улучшение - деление по районам.
- Данные улучшения повысят предсказательную способность модели для разной ценовой категории недвижимости.

