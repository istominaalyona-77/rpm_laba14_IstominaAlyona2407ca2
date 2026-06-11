# Лабораторная работа №14
Тема работы: Управление жизненным циклом DbContext в MVVM-приложениях.
Цель работы: Изучить проблемы, возникающие при некорректном времени жизни DbContext в десктопных MVVM-приложениях (Captive Dependency, утечки памяти). Освоить паттерн IDbContextFactory для управления жизненным циклом контекста базы
данных, научиться работать с отсоединенными сущностями (Detached Entities) и реализовывать безопасные CRUD-операции без утечек состояния Change Tracker.

Варианты решения проблемы долгоживущих ViewModel:
Существует три основных подхода:
  IServiceScopeFactory — создание Scope внутри репозитория. Недостаток: антипаттерн Service Locator, разрушение Unit of Work.
  Scope на уровне ViewModel — ViewModel сама управляет Scope. Недостаток: ViewModel берёт на себя инфраструктурную ответственность.
  IDbContextFactory<TContext> — Фабрика регистрируется как Singleton и создаёт короткоживущие контексты по требованию.

Работа с отсоединёнными сущностями (Detached Entities)
При использовании короткоживущих контекстов сущности, загруженные в одном контексте, становятся отсоединёнными после его уничтожения. Для операции Update необходимо в новом контексте найти сущность по ключу, изменить её значения и вызвать SaveChanges.


<img width="563" height="76" alt="image" src="https://github.com/user-attachments/assets/8031d93e-21fc-4a87-8a36-002d07d0c633" />
Замена на фабрику чтобы всегда работать с актуалтными данными

<img width="534" height="316" alt="image" src="https://github.com/user-attachments/assets/e48f8efa-7d2c-4b6c-a1b2-ef6d80d323e9" />
Read. Используется .AsNoTracking(), чтобы сущности сразу загружались как отсоединённые — это снимает нагрузку с Change Tracker.

<img width="734" height="509" alt="image" src="https://github.com/user-attachments/assets/70870b5b-3fde-48f7-b157-ff4fbe163aa1" />
Create. Метод ExecuteAdd создаёт новый контекст для проверки уникальности номера и сохранения.

<img width="800" height="551" alt="image" src="https://github.com/user-attachments/assets/537dbf32-6a5c-4cda-b854-b37b629f6410" />
Update. Так как сущность из списка теперь отсоединённая, ViewModel хранит только Id контакта. При сохранении реализуется паттерн Fetch-Modify-Save.
Fetch-Modify-Save — это последовательный процесс взаимодействия с данными, который включает три этапа:
1. Извлечение данных из источника (например, базы данных, API или кэша).
2. Модификация полученных данных (изменение значений полей, структуры и т. д.).
3. Сохранение изменённых данных обратно в источник.

<img width="684" height="369" alt="image" src="https://github.com/user-attachments/assets/d61e9986-b8dc-4252-812a-4feee4ddfe81" />
Delete. Метод ExecuteDelete создаёт новый контекст, находит сущность по Id (она автоматически прикрепляется к Change Tracker) и удаляет.




