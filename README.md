1 МОДУЛЬ

Создаем БД, например - milk_factory

Теперь в SQL:

CREATE TABLE customers (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    inn VARCHAR(20),
    address VARCHAR(255),
    phone VARCHAR(20),
    salesman BOOLEAN DEFAULT FALSE,
    buyer BOOLEAN DEFAULT FALSE
);

CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    unit VARCHAR(20) NOT NULL
);

CREATE TABLE materials (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    unit VARCHAR(20) NOT NULL
);

CREATE TABLE specifications (
    id INT AUTO_INCREMENT PRIMARY KEY,
    product_id INT NOT NULL,
    material_id INT NOT NULL,
    quantity DECIMAL(10,3) NOT NULL,

    CONSTRAINT fk_spec_product
        FOREIGN KEY (product_id)
        REFERENCES products(id),

    CONSTRAINT fk_spec_material
        FOREIGN KEY (material_id)
        REFERENCES materials(id)
);

CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    order_date DATE NOT NULL,
    customer_id INT NOT NULL,

    CONSTRAINT fk_order_customer
        FOREIGN KEY (customer_id)
        REFERENCES customers(id)
);

CREATE TABLE order_items (
    id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(10,2) NOT NULL,

    CONSTRAINT fk_item_order
        FOREIGN KEY (order_id)
        REFERENCES orders(id),

    CONSTRAINT fk_item_product
        FOREIGN KEY (product_id)
        REFERENCES products(id)
);


2 МОДУЛЬ

phpMyAdmin → milk_factory → customers → Структура (ПРОСТО ПОСМОТРИМ)

Заказчики.json (таблица customers)

Теперь в SQL добавляем из Заказчики.json:

INSERT INTO customers
(id,name,inn,address,phone,salesman,buyer)
VALUES
('000000001','ООО "Поставка"','','г.Пятигорск','+79198634592',1,1),

('000000002','ООО "Кинотеатр Квант"','26320045123','г. Железноводск, ул. Мира, 123','+79884581555',1,0),

('000000008','ООО "Новый JDTO"','26320045111','г. Железноводсу','+79884581555',1,0),

('000000003','ООО "Ромашка"','4140784214','г. Омск, ул. Строителей, 294','+79882584546',0,1),

('000000009','ООО "Ипподром"','5874045632','г. Уфа, ул. Набережная, 37','+79627486389',1,1),

('000000010','ООО "Ассоль"','2629011278','г. Калуга, ул. Пушкина, 94','+79184572398',0,1);

customers → Обзор (смотрим, что в таблице customers все добавилось)

Цены.xlsx (таблица products)

INSERT INTO products (name, price, unit)
VALUES
('Кефир 2,5% 900г.', 80.00, 'шт'),
('Кефир 3,2% 900г.', 82.00, 'шт'),
('Молоко 2,5% 900г.', 70.00, 'шт'),
('Молоко 3,2% 900г.', 76.00, 'шт'),
('Сметана классическая 15% 540г.', 89.00, 'шт'),
('Сметана классическая 20% 540г.', 92.00, 'шт');

Цены.xlsx (таблица materials)

INSERT INTO materials (name, price, unit)
VALUES
('Молоко нормализованное', 34.00, 'кг'),
('Закваска сметанная', 45.00, 'кг');

Спецификация.xlsx (таблица specifications)

INSERT INTO specifications (product_id, material_id, quantity)
VALUES
(5, 1, 0.900),
(5, 2, 0.070);

Заказ покупателя.xlsx (таблица orders)

ЕСЛИ 

id = 000000010
(если ты поменяла тип id на VARCHAR)

ТО

INSERT INTO orders (order_date, customer_id)
VALUES ('2025-06-06', '000000010');

(таблица order_items)

Открываем products → Обзор

И выполняем:

INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES
(1, 1, 12, 80.00),
(1, 2, 9, 82.00),
(1, 3, 10, 79.00);


3 МОДУЛЬ

НУЖНО ДОБАВИТЬ СПЕЦИФИКАЦИИ
Спецификация — это рецепт продукции.

Например:

Кефир 2,5%
Для производства 1 шт. используется:
0.9 кг молока нормализованного;
0.05 кг закваски.

Кефир 3,2%
0.9 кг молока нормализованного;
0.06 кг закваски.

Молоко 2,5%
1 кг молока нормализованного.
Молоко 3,2%
1.1 кг молока нормализованного.

Сметана 15%
0.9 кг молока нормализованного;
0.07 кг закваски.

Сметана 20%
1 кг молока нормализованного;
0.08 кг закваски.

Посмотрим id материалов

В таблице materials у тебя должно быть:
id Материал
1 Молоко нормализованное
2 Закваска сметанная

Посмотрим id продукции
В таблице products
id Продукция
1 Кефир 2,5% 900г.
2 Кефир 3,2% 900г.
3  Молоко 2,5% 900г.
4 Молоко 3,2% 900г.
5 Сметана классическая 15% 540г.
6 Сметана классическая 20% 540г.

Если запись для сметаны 15% уже существует

У тебя сейчас, скорее всего, есть:

product_id  material_id  quantity
5  1  0.900
5  2  0.070

Их повторно добавлять не нужно.

Тогда можно выполнить такой SQL:

INSERT INTO specifications (product_id, material_id, quantity)
VALUES
-- Кефир 2,5%
(1, 1, 0.900),
(1, 2, 0.050),

-- Кефир 3,2%
(2, 1, 0.900),
(2, 2, 0.060),

-- Молоко 2,5%
(3, 1, 1.000),

-- Молоко 3,2%
(4, 1, 1.100),

-- Сметана 20%
(6, 1, 1.000),
(6, 2, 0.080);

Нужно определить: 
Сколько стоят материалы, которые потребовались для производства заказанной продукции.

ПРИМЕР:
Заказали 1 штуку сметаны 15%.

По спецификации для производства одной сметаны нужно:
Материал | Норма | Цена
Молоко нормализованное | 0.900 кг | 34
Закваска сметанная | 0.070 кг | 45

Стоимость материалов:
0.900 × 34 = 30.60
0.070 × 45 = 3.15
Итого = 33.75

Если заказали 10 штук сметаны:
33.75 × 10 = 337.50

Какие таблицы участвуют?
orders
   ↓
order_items
   ↓
products
   ↓
specifications
   ↓
materials

SELECT
    o.id AS order_id,
    p.name AS product_name,
    oi.quantity AS product_quantity,
    m.name AS material_name,
    s.quantity AS material_quantity,
    m.price AS material_price,
    (oi.quantity * s.quantity * m.price) AS material_cost
FROM orders o
JOIN order_items oi
    ON o.id = oi.order_id
JOIN products p
    ON oi.product_id = p.id
JOIN specifications s
    ON p.id = s.product_id
JOIN materials m
    ON s.material_id = m.id;

Если нужно получить именно полную стоимость заказа по материалам:

SELECT
    o.id AS order_id,
    SUM(oi.quantity * s.quantity * m.price) AS total_material_cost
FROM orders o
JOIN order_items oi
    ON o.id = oi.order_id
JOIN products p
    ON oi.product_id = p.id
JOIN specifications s
    ON p.id = s.product_id
JOIN materials m
    ON s.material_id = m.id
GROUP BY o.id;
