# N
DEP-MGPU
cat > README.md << 'EOF'
# Лабораторная работа Git - Вариант 12

## Студент
- **ФИО:** ДАНСУ КВЕНТИН
- **Группа:** БД-251м
- **Вариант:** 12
- **Дата:** 21.02.2026

## Бизнес-задача
Анализ товарной корзины (Market Basket Analysis) для выявления ассоциативных правил покупок.

## Проектная задача
Реализация алгоритма Apriori для поиска часто встречающихся наборов товаров.

## Техническое задание варианта 12
Использование `git blame` для определения автора строк кода.

## Структура проекта
# Переключение на dev и изменение файла
git checkout dev

cat > src/apriori.py << 'EOF'
"""
Упрощенная версия с использованием библиотеки mlxtend
Для production среды
"""

def find_association_rules_fast(df_transactions, min_support=0.01):
    """
    Быстрый поиск правил с использованием готовой библиотеки
    """
    print(f"Поиск правил с поддержкой {min_support}")
    print("Используется оптимизированная версия")
    return []

if __name__ == "__main__":
    print("Версия для production")
EOF

git add src/apriori.py
git commit -m "feat: production версия с mlxtend"

# Переключение на feature и добавление функционала
git checkout feature/script

cat >> src/apriori.py << 'EOF'

def export_rules_to_csv(rules, filename):
    """
    Экспорт правил в CSV файл
    """
    import csv
    with open(filename, 'w', newline='') as f:
        writer = csv.writer(f)
        writer.writerow(['antecedent', 'consequent', 'support', 'confidence'])
        for rule in rules:
            writer.writerow([
                ','.join(rule['antecedent']),
                ','.join(rule['consequent']),
                rule['support'],
                rule['confidence']
            ])
    print(f"Правила экспортированы в {filename}")
EOF

git add src/apriori.py
git commit -m "feat: добавлен экспорт правил в CSV"

# СЛИЯНИЕ С КОНФЛИКТОМ
git checkout dev
git merge feature/script
# ПОЯВЛЯЕТСЯ КОНФЛИКТ!

# РАЗРЕШЕНИЕ КОНФЛИКТА
cat > src/apriori.py << 'EOF'
"""
Модуль для анализа товарной корзины (Market Basket Analysis)
ОБЪЕДИНЕННАЯ ВЕРСИЯ: полная реализация + быстрый поиск + экспорт

Автор: ДАНСУ КВЕНТИН
Группа: БД-251м
Вариант: 12
"""

from itertools import combinations
from collections import defaultdict
import csv

class Apriori:
    """
    Полная реализация алгоритма Apriori
    """
    
    def __init__(self, min_support=0.1, min_confidence=0.5):
        self.min_support = min_support
        self.min_confidence = min_confidence
        self.frequent_itemsets = []
        self.rules = []
        
    def fit(self, transactions):
        print(f"Обучение Apriori на {len(transactions)} транзакциях")
        
        item_counts = defaultdict(int)
        for transaction in transactions:
            for item in transaction:
                item_counts[item] += 1
        
        n_transactions = len(transactions)
        self.frequent_itemsets = [
            (item,) for item, count in item_counts.items()
            if count / n_transactions >= self.min_support
        ]
        
        self._generate_rules(transactions)
        
    def _generate_rules(self, transactions):
        n_transactions = len(transactions)
        
        for itemset in self.frequent_itemsets:
            if len(itemset) == 1:
                continue
                
            for i in range(1, len(itemset)):
                for antecedent in combinations(itemset, i):
                    consequent = tuple(set(itemset) - set(antecedent))
                    
                    support_antecedent = self._calculate_support(antecedent, transactions)
                    support_both = self._calculate_support(itemset, transactions)
                    
                    if support_antecedent > 0:
                        confidence = support_both / support_antecedent
                        
                        if confidence >= self.min_confidence:
                            self.rules.append({
                                'antecedent': antecedent,
                                'consequent': consequent,
                                'support': support_both,
                                'confidence': confidence
                            })
    
    def _calculate_support(self, itemset, transactions):
        count = 0
        for transaction in transactions:
            if set(itemset).issubset(set(transaction)):
                count += 1
        return count / len(transactions)
    
    def get_rules(self):
        return self.rules
    
    def predict(self, items):
        recommendations = defaultdict(float)
        
        for rule in self.rules:
            if set(rule['antecedent']).issubset(set(items)):
                for item in rule['consequent']:
                    if item not in items:
                        recommendations[item] = max(recommendations[item], rule['confidence'])
        
        return sorted(recommendations.items(), key=lambda x: x[1], reverse=True)
    
    def export_rules_to_csv(self, filename):
        """Экспорт правил в CSV"""
        with open(filename, 'w', newline='') as f:
            writer = csv.writer(f)
            writer.writerow(['antecedent', 'consequent', 'support', 'confidence'])
            for rule in self.rules:
                writer.writerow([
                    ','.join(rule['antecedent']),
                    ','.join(rule['consequent']),
                    f"{rule['support']:.3f}",
                    f"{rule['confidence']:.3f}"
                ])
        print(f"Правила экспортированы в {filename}")


def find_association_rules_fast(df_transactions, min_support=0.01):
    """
    Быстрый поиск правил (для больших данных)
    """
    print(f"Быстрый поиск правил с поддержкой {min_support}")
    print("Примечание: для использования установите mlxtend")
    return []


if __name__ == "__main__":
    print("=" * 60)
    print("ТЕСТИРОВАНИЕ ОБЪЕДИНЕННОЙ ВЕРСИИ APRIORI")
    print("=" * 60)
    
    # Тестовые данные
    test_transactions = [
        ['хлеб', 'молоко', 'масло'],
        ['хлеб', 'молоко'],
        ['хлеб', 'масло'],
        ['молоко', 'масло'],
        ['хлеб', 'молоко', 'масло', 'яйца'],
        ['молоко', 'яйца'],
        ['хлеб', 'яйца']
    ]
    
    # Создание и обучение
    model = Apriori(min_support=0.3, min_confidence=0.6)
    model.fit(test_transactions)
    
    # Вывод правил
    print("\nПравила:")
    for rule in model.get_rules()[:3]:
        print(f"  {rule['antecedent']} -> {rule['consequent']} "
              f"(подд={rule['support']:.2f}, дост={rule['confidence']:.2f})")
    
    # Тест рекомендаций
    print("\nРекомендации для [хлеб, молоко]:")
    for item, conf in model.predict(['хлеб', 'молоко']):
        print(f"  - {item}: {conf:.2f}")
    
    # Тест экспорта
    model.export_rules_to_csv("test_rules.csv")

# Добавление строки от другого разработчика
cat >> src/apriori.py << 'EOF'

def validate_rules(rules, min_confidence=0.5):
    """
    Валидация правил (добавлено Марией Ивановой)
    """
    return [r for r in rules if r['confidence'] >= min_confidence]



# Использование git blame
git blame src/apriori.py > docs/blame_result.txt

# Просмотр результата
cat docs/blame_result.txt
    print("\nБыстрая версия доступна через find_association_rules_fast()")
    print("=" * 60)
EOF

# Завершение разрешения конфликта
cat > src/apriori.py << 'EOF'
"""
Модуль для анализа товарной корзины (Market Basket Analysis)
Алгоритм Apriori для поиска ассоциативных правил

Автор: ДАНСУ КВЕНТИН
Группа: БД-251м
Вариант: 12
"""

from itertools import combinations
from collections import defaultdict

class Apriori:
    """
    Реализация алгоритма Apriori для поиска часто встречающихся наборов товаров
    """
    
    def __init__(self, min_support=0.1, min_confidence=0.5):
        """
        Инициализация алгоритма
        
        Args:
            min_support (float): Минимальная поддержка (0.1 = 10%)
            min_confidence (float): Минимальная достоверность (0.5 = 50%)
        """
        self.min_support = min_support
        self.min_confidence = min_confidence
        self.frequent_itemsets = []
        self.rules = []
        
    def fit(self, transactions):
        """
        Обучение алгоритма на транзакциях
        
        Args:
            transactions (list): Список транзакций (каждая транзакция - список товаров)
        """
        print(f"Обучение Apriori на {len(transactions)} транзакциях")
        print(f"Параметры: min_support={self.min_support}, min_confidence={self.min_confidence}")
        
        # Подсчет частоты отдельных товаров
        item_counts = defaultdict(int)
        for transaction in transactions:
            for item in transaction:
                item_counts[item] += 1
        
        # Фильтрация по минимальной поддержке
        n_transactions = len(transactions)
        frequent_items = [
            (item,) for item, count in item_counts.items()
            if count / n_transactions >= self.min_support
        ]
        
        self.frequent_itemsets = frequent_items
        print(f"Найдено частых наборов: {len(self.frequent_itemsets)}")
        
        # Генерация правил
        self._generate_rules(transactions)
        
    def _generate_rules(self, transactions):
        """
        Генерация ассоциативных правил
        """
        n_transactions = len(transactions)
        
        for itemset in self.frequent_itemsets:
            if len(itemset) == 1:
                continue
                
            for i in range(1, len(itemset)):
                for antecedent in combinations(itemset, i):
                    consequent = tuple(set(itemset) - set(antecedent))
                    
                    # Вычисление поддержки и достоверности
                    support_antecedent = self._calculate_support(antecedent, transactions)
                    support_both = self._calculate_support(itemset, transactions)
                    
                    if support_antecedent > 0:
                        confidence = support_both / support_antecedent
                        
                        if confidence >= self.min_confidence:
                            self.rules.append({
                                'antecedent': antecedent,
                                'consequent': consequent,
                                'support': support_both,
                                'confidence': confidence
                            })
        
        print(f"Сгенерировано правил: {len(self.rules)}")
    
    def _calculate_support(self, itemset, transactions):
        """
        Вычисление поддержки для набора товаров
        """
        count = 0
        for transaction in transactions:
            if set(itemset).issubset(set(transaction)):
                count += 1
        return count / len(transactions)
    
    def get_rules(self):
        """
        Получение сгенерированных правил
        """
        return self.rules
    
    def predict(self, items):
        """
        Предсказание товаров на основе текущей корзины
        
        Args:
            items (list): Товары в корзине
            
        Returns:
            list: Рекомендуемые товары
        """
        recommendations = defaultdict(float)
        
        for rule in self.rules:
            if set(rule['antecedent']).issubset(set(items)):
                for item in rule['consequent']:
                    if item not in items:
                        recommendations[item] = max(recommendations[item], rule['confidence'])
        
        return sorted(recommendations.items(), key=lambda x: x[1], reverse=True)


if __name__ == "__main__":
    # Тестовые данные
    test_transactions = [
        ['хлеб', 'молоко', 'масло'],
        ['хлеб', 'молоко'],
        ['хлеб', 'масло'],
        ['молоко', 'масло'],
        ['хлеб', 'молоко', 'масло', 'яйца'],
        ['молоко', 'яйца'],
        ['хлеб', 'яйца']
    ]
    
    print("=" * 50)
    print("ТЕСТИРОВАНИЕ АЛГОРИТМА APRIORI")
    print("=" * 50)
    
    # Создание и обучение модели
    model = Apriori(min_support=0.3, min_confidence=0.6)
    model.fit(test_transactions)
    
    # Вывод правил
    print("\nСгенерированные правила:")
    for i, rule in enumerate(model.get_rules()[:3], 1):
        print(f"{i}. {rule['antecedent']} -> {rule['consequent']} "
              f"(поддержка={rule['support']:.2f}, достоверность={rule['confidence']:.2f})")
    
    # Тест рекомендаций
    print("\nРекомендации для корзины [хлеб, молоко]:")
    for item, conf in model.predict(['хлеб', 'молоко']):
        print(f"   - {item}: достоверность={conf:.2f}")
    
    print("\n" + "=" * 50)
EOF
git add src/apriori.py
git commit -m "merge: разрешение конфликта - объединение двух версий Apriori"
