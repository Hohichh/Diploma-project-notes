# План доработки текста пояснительной записки

Ниже собран перечень правок по разделам работы. Формулировки приведены в аккуратный рабочий вид, чтобы по этому файлу можно было последовательно редактировать основной текст.

## 3. Реферат

### 3.1 Тема

- Дополнить формулировку темы в соответствии с утвержденным названием работы и общей структурой реферата.
- Проверить, чтобы тема в реферате совпадала с темой на титульном листе и в других служебных элементах документа.

## 4. Перечень условных обозначений

- Просмотреть весь текст работы и собрать используемые аббревиатуры и сокращения.
- Для каждого сокращения добавить краткое и понятное пояснение.
- Убедиться, что в перечень попадают только реально используемые в работе обозначения.

## 5. Введение

### 5.1 Формулировка целевой аудитории

- Заменить выражение «необходимых студентам» на более точную формулировку: «необходимых обучающимся и в частности студентам».

### 5.2 Термин «вертикально-ориентированных»

- Проверить, действительно ли термин «вертикально-ориентированных» нужен в текущем контексте.
- Если он не несет отдельной смысловой нагрузки, заменить его на более простую и прозрачную формулировку.

### 5.3 Конкретизация практической выгоды

- Усилить вывод о полезности системы не только через борьбу с прокрастинацией, но и через более конкретный эффект.
- Возможные акценты: повышение личной организованности обучающихся, снижение нагрузки на кураторов и преподавателей, повышение прозрачности хода учебной деятельности, сокращение потерь времени при работе с долгосрочными задачами.
- Итоговая формулировка должна логично продолжать проблематику введения и не дублировать раздел об экономическом эффекте.

### 5.4 Уточнение предмета исследования

- Проверить корректность определения предмета исследования.
- Сопоставить формулировку предмета с разделом, где рассматриваются исследования по проблематике организации времени и академического планирования.
- При необходимости уточнить предмет так, чтобы он лучше соответствовал реально анализируемой научной базе.

## 6. Аналитический раздел

### 6.1 Формулировка про академическую среду

- В предложении «академическая среда вуза характеризуется слабой структурированностью процессов и высокой потребностью в автономии» заменить слово «вуз» на «высшее учебное заведение» или «университет» в зависимости от общего стиля раздела.

### 6.2 Русификация термина о сроках выполнения

- В предложении о долгосрочных проектах заменить слово «дедлайны» на русскоязычный эквивалент, например «сроки сдачи».

### 6.3 Удаление англоязычной вставки compliance

- В формулировке «Для повышения вовлеченности (compliance) пользователей...» убрать слово в скобках и оставить русский вариант.

### 6.4 Удаление англоязычной вставки loss aversion

- В формулировке про когнитивное искажение «неприятия потери» убрать английский вариант в скобках.

### 6.5 Удаление вставки про комплаенс

- В формулировке «реализация механизмов удержания внимания (комплаенса) и мотивации...» убрать слово в скобках и оставить чистую русскоязычную формулировку.

## 7. Раздел реализации

### 7.1 Переработка подраздела об алгоритме выбора свободного слота

- Переделать подраздел так, чтобы он отражал актуальную реализацию алгоритма размещения подзадач по временным окнам.
- В описании акцент сделать на логике выбора следующего допустимого интервала, учете ограничений пользователя, пропуске выходных и контроле дневного лимита.

### 7.2 Удаление моноширинного шрифта из основного текста

- Убрать из основного текста моноширинное оформление там, где речь идет не о листингах, а об обычных пояснениях.
- Оставить моноширинный шрифт только в листингах, фрагментах кода и при необходимости в строго технических обозначениях.

### 7.3 Вставка актуального алгоритма размещения

- Вставить в раздел актуальный алгоритм размещения подзадач.
- Предпочтительно использовать не полный класс, а либо листинг основного метода, либо псевдокод.
- Подпись к листингу оформить по типу рисунка: по центру снизу.
- Если листинг занимает больше трети страницы, лучше вынести его в приложение `sections/appendix.tex`.
- В пояснениях к листингу меньше ссылаться на конкретные имена переменных; при необходимости переименовывать их словесно на русский язык.

Референсный код текущей реализации:

```java
package io.hohichh.planning_assistant.service.strategy;

import io.hohichh.planning_assistant.model.Subtask;
import io.hohichh.planning_assistant.model.Timeslot;
import io.hohichh.planning_assistant.valueObj.AllocationConstraints;
import io.hohichh.planning_assistant.valueObj.ProposedTimeslot;
import org.springframework.stereotype.Component;

import java.time.DayOfWeek;
import java.time.Duration;
import java.time.Instant;
import java.time.LocalDate;
import java.time.LocalTime;
import java.time.ZoneOffset;
import java.time.ZonedDateTime;
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

@Component
public class GreedySlotAllocationStrategy implements SlotAllocationStrategy {

    private static final LocalTime DAY_START = LocalTime.of(8, 0);
    private static final LocalTime DAY_END = LocalTime.of(22, 0);

    private static final LocalTime MORNING_START = LocalTime.of(8, 0);
    private static final LocalTime MORNING_END = LocalTime.of(12, 0);
    private static final LocalTime AFTERNOON_START = LocalTime.of(12, 0);
    private static final LocalTime AFTERNOON_END = LocalTime.of(18, 0);
    private static final LocalTime EVENING_START = LocalTime.of(18, 0);
    private static final LocalTime EVENING_END = LocalTime.of(22, 0);

    @Override
    public List<ProposedTimeslot> allocate(List<Subtask> subtasks,
                                           List<Timeslot> occupied,
                                           AllocationConstraints constraints) {
        List<OccupiedInterval> intervals = occupied.stream()
                .map(t -> new OccupiedInterval(t.getStartTime(), t.getEndTime()))
                .sorted(Comparator.comparing(OccupiedInterval::start))
                .toList();

        List<ProposedTimeslot> result = new ArrayList<>();
        Instant cursor = constraints.rangeStart();
        Instant rangeEnd = constraints.rangeEnd();
        int minBreak = constraints.minBreakMinutes() != null ? constraints.minBreakMinutes() : 10;
        int dailyBudgetUsed = 0;
        LocalDate currentDay = null;

        for (Subtask subtask : subtasks) {
            int durationMinutes = subtask.getEstimatedMinutes();
            Instant slotStart = findNextAvailableSlot(
                    cursor, durationMinutes, minBreak, intervals, constraints, rangeEnd,
                    dailyBudgetUsed, currentDay
            );

            if (slotStart == null) {
                break;
            }

            Instant slotEnd = slotStart.plus(Duration.ofMinutes(durationMinutes));
            result.add(new ProposedTimeslot(subtask.getId(), slotStart, slotEnd));

            cursor = slotEnd.plus(Duration.ofMinutes(minBreak));

            LocalDate slotDay = slotStart.atOffset(ZoneOffset.UTC).toLocalDate();
            if (currentDay == null || !currentDay.equals(slotDay)) {
                currentDay = slotDay;
                dailyBudgetUsed = durationMinutes;
            } else {
                dailyBudgetUsed += durationMinutes;
            }
        }

        return result;
    }

    private Instant findNextAvailableSlot(Instant cursor, int durationMinutes, int minBreak,
                                          List<OccupiedInterval> intervals,
                                          AllocationConstraints constraints,
                                          Instant rangeEnd,
                                          int dailyBudgetUsed,
                                          LocalDate currentDay) {
        Instant candidate = cursor;
        int maxIterations = 1000;

        for (int i = 0; i < maxIterations; i++) {
            if (!candidate.isBefore(rangeEnd)) {
                return null;
            }

            candidate = snapToWorkingHours(candidate, constraints);
            if (candidate == null || !candidate.isBefore(rangeEnd)) {
                return null;
            }

            if (constraints.avoidWeekends()) {
                candidate = skipWeekends(candidate);
                if (!candidate.isBefore(rangeEnd)) {
                    return null;
                }
            }

            LocalDate candidateDay = candidate.atOffset(ZoneOffset.UTC).toLocalDate();
            int budgetForDay = (currentDay != null && currentDay.equals(candidateDay))
                    ? dailyBudgetUsed : 0;

            if (constraints.maxDailyMinutes() != null
                    && budgetForDay + durationMinutes > constraints.maxDailyMinutes()) {
                candidate = nextDayStart(candidate, constraints);
                continue;
            }

            Instant slotEnd = candidate.plus(Duration.ofMinutes(durationMinutes));
            Instant windowEnd = dayEndInstant(candidate, constraints);
            if (slotEnd.isAfter(windowEnd)) {
                candidate = nextDayStart(candidate, constraints);
                continue;
            }

            Instant conflict = findConflict(candidate, slotEnd, intervals);
            if (conflict != null) {
                candidate = conflict.plus(Duration.ofMinutes(minBreak));
                continue;
            }

            return candidate;
        }

        return null;
    }
}
```

### 7.4 Замена описания про tools на structured output

- Убрать из текста устаревшее описание взаимодействия через tools.
- Вместо этого показать актуальный фрагмент обработки structured output: ожидание ответа модели, извлечение JSON, разбор ответа и переход к формированию черновика.
- Здесь также не нужно приводить слишком большой фрагмент кода: достаточно ключевой вырезки, псевдокода или примера ответа модели.
- Подпись к листингу оформить по центру снизу.
- Если листинг получается слишком объемным, вынести его в приложение `sections/appendix.tex`.

Референсный код текущей реализации:

```java
package io.hohichh.planning_assistant.service.impl;

@Service
public class AgentServiceImpl implements AgentService {

    @Override
    public DraftResponse processRequest(String message, UUID userId, UUID taskId) {
        Task task = taskRepository.findById(taskId)
                .orElseThrow(() -> new EntityNotFoundException("Task not found: " + taskId));

        String scheduleContext = buildScheduleContext(userId, task);
        String semanticContext = buildSemanticContext(userId, task, message);
        String memoryContext = buildMemoryContext(conversationId(userId, taskId));
        String currentDatetime = Instant.now().atOffset(ZoneOffset.UTC)
                .format(DateTimeFormatter.ISO_OFFSET_DATE_TIME);

        LlmPlannerResponse plannerResponse = invokeLlmWithRetry(
                message, taskId, scheduleContext, semanticContext, memoryContext, currentDatetime, task
        );

        List<SubtaskDraft> drafts = toDrafts(plannerResponse.subtasks(), taskId);
        AllocationConstraints constraints = toAllocationConstraints(plannerResponse.constraints(), task);
        List<Timeslot> occupied = timeslotRepository
                .findByUserIdAndStartTimeGreaterThanEqualAndEndTimeLessThanEqual(
                        userId, constraints.rangeStart(), constraints.rangeEnd());
        List<ProposedTimeslot> proposedSlots = allocateSlotsSafely(drafts, occupied, constraints);

        drafts = mergeTimeslots(drafts, proposedSlots);
        return draftService.createDraftBatch(userId, drafts);
    }

    private LlmPlannerResponse invokeLlmWithRetry(...) {
        for (int attempt = 1; attempt <= maxAttempts; attempt++) {
            try {
                String rawResponse = CompletableFuture.supplyAsync(() -> chatClient.prompt()
                                .system(...)
                                .user(message)
                                .call()
                                .content())
                        .get(timeoutMs, TimeUnit.MILLISECONDS);

                return parseResponse(rawResponse);
            } catch (TimeoutException | ExecutionException | RuntimeException e) {
                sleepBackoff(attempt);
            }
        }

        return fallbackPlan(message, task);
    }

    private LlmPlannerResponse parseResponse(String raw) {
        String json = extractJson(raw);
        return objectMapper.readValue(json, LlmPlannerResponse.class);
    }
}
```

### 7.5 Языковое и типографическое выравнивание раздела

- Убрать в этом разделе моноширинное оформление из обычного текста.
- Упоминания статусов, режимов и иных терминов в повествовательной части по возможности переводить на русский язык и оформлять обычным шрифтом `Times New Roman`.
- Англоязычные обозначения оставлять только там, где это действительно требуется для точности, например в листингах и названиях классов.

## 8. Источники

- Удалить лишние источники, не связанные с основным текстом работы.
- После удаления проверить, что все оставшиеся позиции действительно цитируются или содержательно используются в соответствующих разделах.

## Что выполнено в рамках редактирования данного плана

- Перестроена общая структура плана по разделам 3--8.
- Черновые заметки и телеграфные формулировки переписаны в связный рабочий текст.
- Сохранены и аккуратно встроены референсные фрагменты кода для пунктов 7.3 и 7.4.
- Уточнены требования к оформлению листингов, пояснений и приложений.
- Убраны грубые и разговорные формулировки.
- В конце файла добавлена явная отметка о выполненных действиях.