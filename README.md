## Проблема
- В современных финансовых системах миллионы транзакций проходят через платформы, что требует автоматизации их обработки.
- Для каждой транзакции необходимо:
  - Найти подходящего провайдера, который может обработать платеж.
  - Оптимизировать выбор, обеспечивая максимальную прибыль компании.
  - Соблюдать технические и временные ограничения, такие как лимиты провайдеров и их доступность.
## Основные критерии выбора провайдера
1. Доступность провайдера: провайдер должен быть свободен для обработки транзакции в заданное время.
2. Лимиты по сумме: сумма транзакции должна попадать в допустимые границы (LIMIT_MIN и LIMIT_MAX).
3. Оптимизация прибыли: среди доступных провайдеров выбирается тот, который обеспечивает наибольшую прибыль.

## Построение таргета (ранга)
1. Цель ранжирования:
    - Упорядочить провайдеров по степени предпочтительности для обработки транзакций на основе вычисленной метрики "прибыль" (profit).

2. Этапы построения таргета:

    - Для каждой транзакции вычисляется прибыль как разница между суммой транзакции и комиссией провайдера.
    - Провайдеры, прошедшие фильтры (доступность, лимиты, актуальность), получают оценку прибыли.
    - Список провайдеров сортируется по убыванию прибыли.

3. Формирование таргета (ранга):

    - Провайдер с максимальной прибылью получает ранг 1.
    - Последующие провайдеры ранжируются в порядке убывания прибыли (2, 3, и т.д.).
    - Если провайдеры имеют одинаковую прибыль, они получают одинаковый ранг (разрешение конфликтов зависит от бизнес-логики).
4. Результат:

    - Ранжированный список провайдеров, готовый для выбора наиболее выгодного варианта.
5. Особенности:

    - Если провайдеры не соответствуют минимальным требованиям (например, по лимитам или валюте), таргет не формируется, возвращается None.
  
## CatBoost Ranker:
1. Что такое CatBoost Ranker:

    - Это алгоритм, предназначенный для решения задач ранжирования, основанный на градиентном бустинге над решающими деревьями.

2. Оптимизируемая метрика:

    - Обычно используется DCG (Discounted Cumulative Gain) или её нормализованная версия NDCG:
      - Метрика оценивает качество ранжирования, учитывая позицию объекта в списке (чем выше объект, тем больший вклад в метрику).
      - Учёт убывания "ценности" позиции с увеличением ранга (логарифмическое снижение веса).

3. Как это работает:
  
    - Модель обучается предсказывать ранги:
      - Для каждой транзакции обучается упорядочивать провайдеров по предсказанным значениям (логарифм прибыли, вероятность успеха и т.д.).
    - Группировка данных:
      - Данные передаются в виде групп (группой является транзакция с её провайдерами).
    - Потери (loss function):
      - Оптимизация ранжирования осуществляется через функцию потерь, например, YetiRank или PairwiseLogit:
        - YetiRank учитывает порядок объектов и разницу в значениях между ними.
        - PairwiseLogit минимизирует вероятность неправильного упорядочивания пар.
4. Результат:

    - Модель предсказывает значения, которые упорядочиваются для формирования итогового ранга провайдеров.
    - Оптимизация нацелена на повышение метрики NDCG, что приводит к выбору лучших провайдеров для транзакции.