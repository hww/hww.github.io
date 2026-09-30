# 📑 Технический стандарт: Архитектура и законы игрового мира

------------------------------

## 📜 ЧАСТЬ I. АРХИТЕКТУРНЫЙ МАНИФЕСТ## Два правила «живого» движка## 1. Подписывайся на смыслы, а не на имена (Принцип отчуждения)

Мы никогда не привязываем логику к конкретному объекту в памяти. Связывать системы напрямую — значит плодить баги и путаться в порядке их создания.

* Как мы делаем: Мы подписываемся на Тип сообщения и Тег (роль) объекта.
* Почему это работает: Скрипту всё равно, создан ли уже объект. Он просто ждет сигнал из нужного канала. Если объект еще не создан или уже погиб — система не упадет и код не сломается.

## 2. Данные — мертвы, Системы — вдыхают жизнь (Принцип пассивной сцены)

Объекты в редакторе мира — это просто «тупые» коробки со свойствами. Они статичны и не имеют собственного разума или функций обновления.

* Как мы делаем: При запуске сцены данные лежат неподвижно, пока не запустится внешний управляющий скрипт (Менеджер Систем). Именно он берет эти пассивные данные, связывает их между собой и запускает логику.
* Почему это работает: Мы получаем полный контроль над порядком выполнения систем, игра не тормозит из-за тысяч мелких скриптов, а делать сохранения (Save/Load) становится невероятно просто.

------------------------------

## 📜 ЧАСТЬ II. СОГЛАШЕНИЕ О СТРУКТУРЕ МИРА## Гибридный Data-Driven подход (Опыт Naughty Dog & Guerrilla Games)## 1. Концепция сущностей и именования

* Плоский реестр сцены (Entity Registry): На уровне выполнения (runtime) сцена всегда представляет собой абсолютно плоский список уникальных сущностей (Entities). Никаких вложенных деревьев трансформаций в логическом контуре.
* Строковое Длинное Имя (Full Unique Name): Каждый объект обязан иметь текстовое уникальное имя, гарантируемое редактором (авто-инкремент индексов при дублировании: gate-1, gate-2).
* Контекстный префикс: Если объекты создаются внутри подсцены, редактор автоматически добавляет префикс контейнера к имени объекта: [Название_Зоны]/[Имя_Объекта] (например, swamp_ambush/part-spawner-1). Это позволяет копировать зоны целиком без конфликта имен.

## 2. Контур связей (Кросс-референсы)

* Связывание по Явным Текстовым Именам: Все перекрестные ссылки между объектами в редакторе указываются строго в виде текстовых строк, содержащих полное уникальное имя целевого объекта (например: lock-name = "gate-lock-1").
* Рантайм-разрешение (Lazy Resolution): Объекты не хранят жесткие ссылки друг на друга в памяти. При активации управляющего скрипта система делает один текстовый запрос в плоский список. Если объект с таким именем еще не загружен — система просто ждет его стриминга, не ломая игру.
* Слепое переименование (Редакторский хук): При переименовании объекта в плоском списке, редактор сканирует текстовые поля свойств остальных объектов и автоматически обновляет строку.

## 3. Разделение Иерархий (Понятийный контур)

Движок жестко разделяет три вида «деревьев», которые в коммерческих движках часто спутаны в одну кучу:

   1. Дерево Проекта (Levels / Assets Tree): Структура папок на жестком диске. Отвечает только за то, в каком файле на диске лежат настройки зоны.
   2. Дерево Трансформаций (Transform Hierarchy): Физическая привязка (Parent/Child). Нужна только для матриц перемещения (колесо крутится за машиной). Сущности в логическом списке при этом лежат раздельно.
   3. Логическое Дерево (Отсутствует): В логике сцены дерева нет. Все сущности равны и общаются через плоскую шину данных.

------------------------------

## 📜 ЧАСТЬ III. КОНТУР ДИНАМИЧЕСКИХ ДАННЫХ (TAGS & FACTS)## 1. Принцип Динамических Свойств (Dynamic Property Overlay)

* Любая сущность на сцене имеет жесткую базовую схему (Schema), определяющую только фундаментальные данные (имя, трансформ, базовый скрипт).
* Каждая сущность имеет динамический JSON-контейнер (TAGS / Dynamic Fields), работающий как оверлей данных (speed = 20, color = "red") без изменения кода и перекомпиляции схем редактора.
* Управляющие скрипты взаимодействуют с объектом через "мягкое" чтение этих полей (методы типа GetTagString или GetTagFloat). Если поля в JSON нет — скрипт использует дефолтное значение.

## 2. Сравнение UX-подходов к редактированию динамических данных

* Подход Naughty Dog (TAGS) — Текстовый минимализм: Одно открытое поле ввода, куда дизайнер вручную пишет JSON-структуру. Максимальная гибкость, легкий Merge в Git, но высокий риск опечаток.
* Подход Guerrilla Games (FACTS) — Визуальный GUI: Полноценный табличный интерфейс (списки, выпадающие меню типов, плюсики). Исключает синтаксические ошибки дизайнера, наглядно отображает типы (Vector3, Color), но требует написания кастомного кода редактора.

## 3. Техническое решение: Компромиссная Unity-реализация (DynamicFacts)

Для достижения минимализма ND в коде и наглядности GUI в рантайме применяется класс DynamicFacts. Через интерфейс ISerializationCallbackReceiver он автоматически преобразует сырую JSON-строку из инспектора в быстрый типизированный словарь для Систем:

```cs
namespace RA.Runtime
{
    [Serializable]
    public class DynamicFacts : BaseFacts, ISerializationCallbackReceiver
    {
        [SerializeField, TextArea(3, 20)]
        private string jsonData = "{}"; // Text field input (Naughty Dog TAGS style)

        private Dictionary<string, string> facts = new Dictionary<string, string>(); // Fast runtime registry

        // Unity serialization processing via single-pass tokenization
        public void OnBeforeSerialize() => jsonData = ConvertToJsonString();
        public void OnAfterDeserialize() => facts = ParseJsonStringToDictionary(jsonData);

        protected override bool TryGetInternal<T>(string name, out T value)
        {
            value = default;
            if (facts.TryGetValue(name, out string jsonValue) && !string.IsNullOrEmpty(jsonValue))
            {
                value = ConvertFromJson<T>(jsonValue); // Safe types resolution (int, float, Vector3, Color)
                return true;
            }
            return false;
        }

        protected override void SetInternal<T>(string name, T value) => facts[name] = ConvertToJson(value);
        protected override bool RemoveInternal(string name) => facts.Remove(name);
        protected override bool ContainsInternal(string name) => facts.ContainsKey(name);
        protected override void ClearInternal() => facts.Clear();
    }
}
```

------------------------------

## 📜 ЧАСТЬ IV. АСИНХРОННЫЙ СТРИМИНГ И АКТОРЫ (СЦЕНЫ-КОНТЕЙНЕРЫ)

Мир не является единым монолитным списком. Он состоит из Глобальной сцены (Мастер-мира) и динамически накладываемых Подсцен (Контейнеров).

* Скрытие в иерархии (Актёры): Интерактивные объекты на сцене оформляются в виде пассивных C#-компонентов Actor. Они хранят текстовое длинное имя (DisplayName) и оверлей данных (DynamicFacts), но полностью лишены метода Update(). Они неподвижны до команды скрипта.
* Локальная область видимости (Local Scope): Объекты внутри такого контейнера сгруппированы по смыслу (квест, схватка, лагерь) и видят друг друга по коротким относительным именам. Скрипт зоны изолирован от соседей по карте.
* Жизненный цикл (Директор Загрузки): Глобальный C#-менеджер ZoneManager асинхронно подгружает сцену в память. Он сканирует иерархию, регистрирует всех Actor в плоском списке и запускает Локального Lua-Директора (текстовый скрипт с диска). При выходе из зоны вся область видимости Lua стирается, а в долгоживущей таблице фактов сохраняется лишь текстовая запись-достижение ("swamp_clear" = true), защищая память от утечек.

------------------------------

## 📜 ЧАСТЬ V. МАНИФЕСТ СКРИПТОВОГО КОНТУРА## Взаимодействие миров: Кто главный в игре?## 1. Первопричина: Отказ от Юнити-центричности

Мы полностью уничтожаем хаос, в котором каждый гейм-объект живет своей жизнью и тянет спагетти-связи к соседям. Мы отказываемся от логики «умных объектов» (подход OnEnable/Update), которая делает сцену неуправляемой.

## 2. Источник правды: Lua-Директор сверху (Top-Down)

В нашей архитектуре Скрипт — это Постановщик Трагедии, а C#-объекты сцены — пассивные марионетки.

* Не сцена и её объекты управляют логикой.
* Централизованный Скрипт-Директор загружает сцену, сам решает, куда телепортировать игрока, находит пассивные сущности по текстовым именам и активирует их конвейеры (спаунеры, триггеры) по мере развития сценария.

## 3. Архитектура пространств имен (Изолированные стейты)

Чтобы память не засорялась, а переменные разных квестов не конфликтовали, мы разделяем мир скриптов на два уровня:

* Глобальный Стейт (Один на всю игру): Живет постоянно. Управляет глобальной шиной данных (Global Facts), сохранениями и переключением уровней.
* Локальный Стейт Зоны (Свой у каждого Scene Instance): Рождается строго в момент асинхронной активации игровой зоны и полностью стирается из памяти при выходе из нее. Локальные скрипты полностью изолированы в своей области видимости.

------------------------------

## 📜 ЧАСТЬ VI. ВЫСОКОПРОИЗВОДИТЕЛЬНЫЙ LUA-DCS МОСТ## 1. Передача типов по ID (Zero String Lookups)

В рантайме на горячих путях (hot paths) строки не используются. При старте игры глобальный скрипт Lua один раз опрашивает C#-реестр пулов компонентов и строит таблицу числовых констант COMPONENT.Имя = ID. Все обращения из Lua к ядру DCS происходят строго через unboxed-числа. Аллокация компонентов в C#-пулах выполняется по прямому индексу массива за O(1).

## 2. Перманентные подписки классов (No-Boilerplate Events)

Объекты в Lua не подписываются на события динамически при смене стейтов. При создании экземпляра объекта (в его базовом абстрактном классе Process) автоматический итератор сканирует таблицу его состояний, находит все функции формата on_event_[ИмяСобытия] и регистрирует их в C# пуле подписок EventSubscription один раз на весь жизненный цикл объекта. Контекстное поведение (игнорирование или обработка) настраивается подменой локальных Lua-таблиц состояний, не нагружая C#.

## 3. Двухфазная доставка и извлечение данных (Unboxed Payload)

События в DCS — это короткоживущие (1 кадр) компоненты данных на стороне отправителя. При срабатывании C# EventSystem пробрасывает в Lua-роутер только два unboxed-числа: HostID получателя и DenseIndex самого компонента события в пуле. Роутер находит объект в Lua по ID за O(1). Если объекту требуются числовые/векторные данные события, он точечно вытягивает их из C# по индексу через метод getData() без аллокаций мусора в куче.

## 4. Асинхронные потоки состояний (Coroutine Threading)

Чтобы избавить дизайнеров от написания покадровых счетчиков времени в циклах Update, базовый абстрактный класс Process имеет встроенную поддержку корутин Lua. Если состояние объявляет метод co_update, он изолируется в поток. Это позволяет писать сложную последовательную логику (паузы, ожидания анимаций) в виде линейного текста с командами coroutine.yield(), замораживающими выполнение строго на 1 игровой кадр

------------------------------

## 📜 ЧАСТЬ VII. ТЕХНИЧЕСКИЙ КОД СТРИМИНГА И АКТОРОВ## 1. Пассивная марионетка сцены (Actor.cs)

```cs
using UnityEngine;

namespace DynamicComponent.Lua
{
    /// <summary>
    /// A passive scene object representation. Acts as a data container anchor 
    /// and a bridge between Unity's hierarchy and the Data-Oriented dynamic core.
    /// </summary>
    [DisallowMultipleComponent]
    public class Actor : MonoBehaviour
    {
        [Header("Entity Metadata")]
        [Tooltip("The unique long string name of this object, validated by the editor environment.")]
        [SerializeField] private string displayName;

        [Header("Dynamic Properties")]
        [Tooltip("The JSON-backed dynamic overlay data container (Naughty Dog TAGS style).")]
        [SerializeField] private DynamicFacts dynamicFacts;

        public string DisplayName => displayName;
        public DynamicFacts Facts => dynamicFacts;

        private void OnValidate()
        {
            if (string.IsNullOrEmpty(displayName))
            {
                displayName = gameObject.name.ToLower().Replace(" ", "_");
            }
        }
    }
}
```

## 2. Директор Асинхронной Загрузки (ZoneManager.cs)

```cs
using System;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

namespace DynamicComponent.Lua
{
    /// <summary>
    /// Global manager responsible for asynchronous scene streaming, local scope allocation,
    /// and routing world entities from Unity subscenes straight into the script engine layer.
    /// </summary>
    public class ZoneManager : MonoBehaviour
    {
        private static ZoneManager _instance;
        public static ZoneManager Instance => _instance;

        private readonly Dictionary<string, Actor> _activeActors = new Dictionary<string, Actor>(StringComparer.OrdinalIgnoreCase);
        private readonly Dictionary<string, ZoneScriptContext> _activeZones = new Dictionary<string, ZoneScriptContext>(StringComparer.OrdinalIgnoreCase);

        void Awake()
        {
            if (_instance != null && _instance != this)
            {
                Destroy(gameObject);
                return;
            }
            _instance = this;
            DontDestroyOnLoad(gameObject);
        }

        public void LoadZoneAsync(string sceneName, string luaScriptRelativePath)
        {
            if (_activeZones.ContainsKey(sceneName))
            {
                Debug.LogWarning($"[ZoneManager] Zone scene '{sceneName}' is already loaded.");
                return;
            }

            var asyncLoad = SceneManager.LoadSceneAsync(sceneName, LoadSceneMode.Additive);
            asyncLoad.completed += (operation) =>
            {
                InitializeLoadedZone(sceneName, luaScriptRelativePath);
            };
        }

        private void InitializeLoadedZone(string sceneName, string luaScriptRelativePath)
        {
            Scene loadedScene = SceneManager.GetSceneByName(sceneName);
            if (!loadedScene.IsValid()) return;

            GameObject[] rootObjects = loadedScene.GetRootGameObjects();
            List<Actor> zoneActors = new List<Actor>();

            foreach (var root in rootObjects)
            {
                var actorsInRoot = root.GetComponentsInChildren<Actor>(true);
                zoneActors.AddRange(actorsInRoot);
            }

            foreach (var actor in zoneActors)
            {
                if (!_activeActors.ContainsKey(actor.DisplayName))
                {
                    _activeActors[actor.DisplayName] = actor;
                }
            }

            Func<string, BaseFacts> registryLookupDelegate = (entityName) =>
            {
                if (_activeActors.TryGetValue(entityName, out Actor actor)) return actor.Facts;
                return null;
            };

            var zoneContext = new ZoneScriptContext(sceneName, registryLookupDelegate);
            _activeZones[sceneName] = zoneContext;

            string scriptCode = LuaManager.Instance.ReadScriptFile(luaScriptRelativePath);
            if (!string.IsNullOrEmpty(scriptCode))
            {
                zoneContext.RunDirectorScript(scriptCode);
            }
        }

        public void UnloadZoneAsync(string sceneName)
        {
            if (_activeZones.TryGetValue(sceneName, out ZoneScriptContext context))
            {
                context.Dispose();
                _activeZones.Remove(sceneName);

                Scene sceneToUnload = SceneManager.GetSceneByName(sceneName);
                if (sceneToUnload.IsValid())
                {
                    foreach (var root in sceneToUnload.GetRootGameObjects())
                    {
                        foreach (var actor in root.GetComponentsInChildren<Actor>(true))
                        {
                            _activeActors.Remove(actor.DisplayName);
                        }
                    }
                    SceneManager.UnloadSceneAsync(sceneToUnload);
                }
            }
        }
    }
}
```

