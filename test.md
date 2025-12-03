graph TD
    %% Определение стилей
    classDef system fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef human fill:#fff9c4,stroke:#fbc02d,stroke-width:2px;
    classDef startend fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px,rx:10,ry:10;

    %% Начало
    Start((Начало процесса)):::startend --> Step1[Выгрузка номенклатуры из док. Импорта<br/>(без тех. описаний)]:::system
    
    %% Основной поток
    Step1 --> Step2[Генерация технических описаний]:::system
    Step2 --> Step3[Запись описаний в базу УПП]:::system
    
    %% Разделение потоков
    Step3 --> Fork{Параллельный<br/>шлюз}
    
    %% Ветка 1: Обучение и Регламенты
    Fork --> Branch1_1[Запрос регламента у отдела BRP]:::human
    Branch1_1 --> Branch1_2[Адаптация регламента под другие отделы]:::human
    Branch1_2 --> Branch1_3[Установка блокировки в УПП<br/>на проведение 'Заказ поставщику'<br/>без тех. описания]:::system
    Branch1_3 --> Branch1_4[Обучение менеджеров<br/>новому регламенту]:::human
    
    %% Ветка 2: Работа с сайтом
    Fork --> Branch2_1[Парсинг каталогов<br/>(получение HTML описаний)]:::system
    Branch2_1 --> Branch2_2[Настройка обмена УПП -> УТ -> Сайт 1]:::system
    Branch2_1 --> Branch2_3[Настройка обмена УПП -> Сайт 2]:::system
    
    %% Детали настройки обмена (комментарии/ассоциации)
    Branch2_2 -.-> Note1[Требование:<br/>Корректное отображение описания +<br/>Скрытое тех. описание для ботов]
    Branch2_3 -.-> Note1
    
    %% Слияние потоков
    Branch1_4 --> Join{Слияние}
    Branch2_2 --> Join
    Branch2_3 --> Join
    
    %% Конец
    Join --> End((Конец процесса)):::startend