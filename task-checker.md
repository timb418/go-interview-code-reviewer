В рамках подготовки к собеседованиям на бэкенд-разработчика Go я практикуюсь в решении задач, 
которые часто встречаются на технических интервью. Это задания формата "анализ кода" 
(вывод, ошибки, рефакторинг).
Твоя цель - проверить постановку задач ниже. 
Ниже - некоторые правила, на которые тебе надо ориентироваться.

Ниже представлен формальный набор правил для проверяльщика, цель которого — обеспечить корректность постановки задач формата «анализ кода» в контексте технических интервью на позицию бэкенд-разработчика на Go. Эти правила помогут гарантировать, что задачи задаются однозначно, содержат минимальное необходимое описание и не включают подсказки, способные упростить решение для кандидата.

---

### 1. Отсутствие явных подсказок

- **Запрещённые комментарии с подсказками:**  
  В исходном коде не допускается наличие комментариев, явно указывающих на последствия определённых действий (например, `"// Эта строка вызовет панику"`).  
- **Нейтральные комментарии:**  
  Если комментарии используются для структурирования или пояснения синтаксиса, они должны быть максимально нейтральными и не должны намекать на логику исправления или указывать, что именно является ошибкой.

---

### 2. Компактность и однозначность постановки задачи

- **Ясное описание проблемы:**  
  Задача должна быть сформулирована так, чтобы кандидат однозначно понимал, что требуется сделать. Описание должно избегать двусмысленностей, но не раскрывать решения.
- **Минимальное количество вспомогательной информации:**  
  Необходимо включать только те детали, которые нужны для понимания контекста. Лишняя информация, которая может дать подсказки или намеки, должна быть исключена.

---

### 3. Стиль и форматирование кода

- **Соответствие стилю Go:**  
  Код должен быть оформлен в соответствии с общепринятыми стандартами и рекомендациями для Go, без изменений, которые могут указывать на неверные или нестандартные подходы.
- **Отсутствие «подсказочных» структур:**  
  Никакие конструкции или комментарии к коду не должны давать понять, что определённое решение или подход является предпочтительным. Вся логика должна быть представлена таким образом, чтобы кандидат самостоятельно проводил анализ.

---

### 4. Точность тестовых примеров (если применимо)

- **Корректные и независимые тесты:**  
  Если к задаче прилагается код с тестовыми примерами, они должны быть сконструированы таким образом, чтобы не давать прямых указаний на причину возникновения ошибки или на порядок исправления.
- **Разделение тестового кода и логики:**  
  Тесты и вспомогательный код (например, функции тестирования) не должны пересекаться с основным кодом решения, чтобы исключить возможность получить подсказку по решению.

---

### 5. Контроль за скрытыми подсказками

- **Автоматический анализ комментариев:**  
  Проверяльщик должен искать шаблоны комментариев, содержащих явно выраженные указания («подсказки») на возможные ошибки, даже если они не являются прямыми комментариями к строкам кода.  
- **Проверка описания задачи:**  
  Текст задания (описание проблемы, цели, требований) не должен содержать указаний, предполагающих конкретные варианты реализации или объяснений, которые облегчают принятие решения кандидатом.

---

### 6. Дополнительные рекомендации

- **Проверка на избыточность:**  
  Все элементы задачи (описание, код, тесты) должны оцениваться на предмет избыточной информации. Если какой-либо фрагмент кода или пояснения дублируют содержимое, которое может указывать на верное решение, его следует убрать или переформулировать.
- **Контекст задачи:**  
  Задача должна имитировать реальную ситуацию, встречающуюся на собеседовании, с минимальным набором деталей, гарантируя, что кандидат будет вынужден самостоятельно проводить анализ и искать оптимальное решение.

---

Ниже задачи, которые тебе надо проверить. Если задача соответствует правилам, просто выведи изначальный вариант в том же виде. Если требует изменений, внеси их и выведи только исправленную версию. Выводи сразу в похожем формате, что и были описаны задачи, просто с правками, где надо. В конце можно дать саммари, где нужны были правки и какие. Ничего лишнего.
