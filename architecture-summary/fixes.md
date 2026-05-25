1. поменять часть системного промпта в приложении:
```
**[OUTPUT FORMAT]**
Ты должен вернуть только строго валидный JSON без пояснений до или после него. JSON должен соответствовать следующей структуре structured output:

{
  "taskTitle": "Название глобальной задачи (если создается с нуля) или null",
  "subtasks": [
    {
      "title": "string",
      "estimatedMinutes": 45,
      "cognitiveLoad": "HIGH"
    }
  ],
  "constraints": {
    "deadlineSemantic": "END_OF_WEEK",
    "deadlineRawText": "к пятнице",
    "priority": "HIGH",
    "preferredTimeOfDay": "ANY",
    "avoidWeekends": false,
    "maxDailyMinutes": null
  }
}

Требования к полям:
- `taskTitle` --- краткое название всей задачи целиком. Если `{TASK_ID_OR_NULL}` не равно null (задача уже существует), верни null.
- `subtasks` --- непустой массив подзадач.
- `title` --- краткое конкретное действие.
- `estimatedMinutes` --- целое число от 25 до 90.
- `cognitiveLoad` --- одно из значений `HIGH`, `MEDIUM`, `LOW`.
- `constraints.deadlineSemantic` --- выбери одно из: `TODAY`, `TOMORROW`, `END_OF_WEEK`, `NEXT_WEEK`, `END_OF_MONTH`, `CUSTOM_DATE`, `NONE`.
- `constraints.deadlineRawText` --- точная фраза пользователя, указывающая на срок (например, "до 15 мая", "на выходных"). Если срока нет, верни null.
- `constraints.priority` --- `HIGH` (срочно, до 48ч), `MEDIUM` (до недели), `LOW` (далеко или не указано).
- `constraints.preferredTimeOfDay` --- `MORNING`, `AFTERNOON`, `EVENING` или `ANY`.
- `constraints.avoidWeekends` --- `true` или `false`.
- `constraints.maxDailyMinutes` --- число или `null`.

Запрещено возвращать любые другие поля.
```
а также в коде планера в папке resources

2. Нужно обновить код алгоритма обработки запроса. а именно часть где мы извлекаем задачу по айди. смысл в том, что теперь при отсутсвии задачи в системе нужно чтобы она создавалась  и модель назначала - `constraints.deadlineSemantic` --- выбери одно из: `TODAY`, `TOMORROW`, `END_OF_WEEK`, `NEXT_WEEK`, `END_OF_MONTH`, `CUSTOM_DATE`, `NONE`. а не считала сама.

«Нам нужно отрефакторить логику расчета дедлайнов и создания задач, чтобы строго следовать нейросимволическому подходу. LLM больше не должна вычислять точную дату (ISO-8601), это будет делать Java.

Обнови DTO AgentRequest, убедись, что taskId опционален (может быть null).

Обнови классы, описывающие ответ LLM (например, LlmPlannerResponse, ConstraintsDTO). Удали deadlineIso. Добавь строковое поле taskTitle на верхний уровень. В ConstraintsDTO добавь deadlineSemantic (Enum или String) и deadlineRawText (String).

В AgentService реализуй метод calculateDeadline(String semantic, String rawText), который берет LocalDateTime.now() и через switch по deadlineSemantic (TODAY, TOMORROW, END_OF_WEEK, NEXT_WEEK, END_OF_MONTH, CUSTOM_DATE, NONE) высчитывает точный дедлайн.

В AgentService обнови логику: если request.taskId() == null, создай новую сущность Task с названием из taskTitle и дедлайном из calculateDeadline. Сохрани её в БД, а затем привяжи к ней сгенерированные подзадачи. Если taskId != null, просто достань существующую Task из БД и привяжи подзадачи к ней.»

### сценарии для текста:

Сценарий А: Свободный текстовый запрос (Создание на лету)

Действие: Отправляем POST /agent/generate с телом {"message": "Распланируй мне курсовую по ООП до конца месяца"} (поле task_id отсутствует).

Ожидаемое поведение сервера:

LLM возвращает JSON, где taskTitle = "Курсовая работа по ООП", а deadlineSemantic = "END_OF_MONTH".

Java вычисляет последний день текущего месяца.

Java создает новую запись в таблице tasks (Курсовая работа по ООП).

Java генерирует подзадачи (Draft) и привязывает их к новому ID задачи.

Проверка: В БД появилась новая задача, в ответе API пришли подзадачи.

Сценарий Б: Уточнение существующей задачи (Гибридный UI)

Действие: Выбираем в UI существующую задачу "Лабораторная №1" (допустим, ID = 123e4567-e89b...). Пишем в чат: "Разбей ее на 3 шага по 40 минут". Отправляем POST с телом {"message": "Разбей ее...", "task_id": "123e4567..."}.

Ожидаемое поведение сервера:

Сервер находит задачу в БД и передает её контекст в LLM.

LLM возвращает JSON, где taskTitle = null (по инструкции).

Java видит task_id, игнорирует taskTitle от LLM и использует существующую задачу.

Java генерирует подзадачи и привязывает их к старому task_id.

3. нужно посмотреть соотвествут ли псевдокод алгоритма декомпзиции задач текущей обновленной логике и учитывает кейсы наличия taskid в базе и его отсутвия. отредактировать его.

4. в пункте где описывается Псевдокод алгоритма планирования временных слотов, нужно дополнить абзац текстом об этих двух граничных случаях и согласовать с текстом о декомпозиции задач