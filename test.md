graph TD
    %% Определение дорожек (Swimlanes) через подграфы
    subgraph "Роль: Оператор / Склад (Действия людей)"
        StartEvent((Начало: Физический заказ получен))
        Task_Assembly[Задача: Выполнить операции по сборке]
    end

    subgraph "Роль: Система (Автоматические действия)"
        Task_CreateSO[Сервис-задача: Создать 'Заказ покупателя']
        Task_CreatePL[Сервис-задача: Создать 'Упаковочный лист']
        Task_UpdatePL_Status[Сервис-задача: Изменить статус 'Упаковочного листа' на 'Собран']
        Task_UpdateSO_Status[Сервис-задача: Изменить статус 'Заказа покупателя' на 'Ожидает отгрузку']
        Task_CreateImpl[Сервис-задача: Создать 'Реализацию ТМЗ и услуг']
        Task_NotifyManager[Отправка сообщения: Уведомить менеджера с вложениями]
    end

    subgraph "Роль: Менеджер"
        Event_ReceiveNotify((Получено уведомление и документы))
        EndEvent((Конец: Заказ готов к отгрузке))
    end

    %% Определение Объектов Данных (Документов и их статусов)
    DataObject_SO_Assembly[/"Док: Заказ покупателя<br>[Статус: Сборка]"/]
    DataObject_PL_Awaiting[/"Док: Упаковочный лист<br>[Статус: Ожидает сборки]"/]
    DataObject_PL_Assembled[/"Док: Упаковочный лист<br>[Статус: Собран]"/]
    DataObject_SO_Shipping[/"Док: Заказ покупателя<br>[Статус: Ожидает отгрузку]"/]
    DataObject_Impl[/"Док: Реализация ТМЗ"/]
    DataObject_Waybill[/"Док: Товарная накладная"/]

    %% --- Поток управления (Стрелки) ---

    %% 1. Старт и создание первичных документов
    StartEvent --> Task_CreateSO
    Task_CreateSO -- Создает --> DataObject_SO_Assembly
    Task_CreateSO --> Task_CreatePL
    Task_CreatePL -- Создает на основании ЗП --> DataObject_PL_Awaiting

    %% 2. Передача на склад и сборка
    Task_CreatePL --> Task_Assembly
    DataObject_PL_Awaiting -. Входные данные .-> Task_Assembly

    %% 3. Завершение сборки и автоматическая цепочка обновлений
    Task_Assembly -- Фиксация завершения сборки --> Task_UpdatePL_Status
    Task_UpdatePL_Status -- Обновляет статус --> DataObject_PL_Assembled
    
    Task_UpdatePL_Status --> Task_UpdateSO_Status
    Task_UpdateSO_Status -- Обновляет статус --> DataObject_SO_Shipping

    %% 4. Создание финальных документов
    Task_UpdateSO_Status --> Task_CreateImpl
    Task_CreateImpl -- Создает --> DataObject_Impl
    Task_CreateImpl -- Создает --> DataObject_Waybill

    %% 5. Уведомление менеджера
    Task_CreateImpl --> Task_NotifyManager
    Task_NotifyManager -- Отправка реализации и накладной --> Event_ReceiveNotify
    
    %% 6. Завершение
    Event_ReceiveNotify --> EndEvent

    %% Стилизация для наглядности (Mermaid специфично)
    style DataObject_SO_Assembly fill:#fff,stroke:#333,stroke-width:2px,stroke-dasharray: 5 5
    style DataObject_PL_Awaiting fill:#fff,stroke:#333,stroke-width:2px,stroke-dasharray: 5 5
    style DataObject_PL_Assembled fill:#e6ffe6,stroke:#333,stroke-width:2px
    style DataObject_SO_Shipping fill:#e6ffe6,stroke:#333,stroke-width:2px
    style DataObject_Impl fill:#e6ffe6,stroke:#333,stroke-width:2px
    style DataObject_Waybill fill:#e6ffe6,stroke:#333,stroke-width:2px
    style subgraph "Роль: Система (Автоматические действия)" fill:#f0f8ff,stroke:#87cefa
    style subgraph "Роль: Оператор / Склад (Действия людей)" fill:#fff5e6,stroke:#ffa500
