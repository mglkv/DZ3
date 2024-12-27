# DZ3
## Задание №3
Разработать инструмент командной строки для учебного конфигурационного
языка, синтаксис которого приведен далее. Этот инструмент преобразует текст из
входного формата в выходной. Синтаксические ошибки выявляются с выдачей
сообщений.
Входной текст на учебном конфигурационном языке принимается из
файла, путь к которому задан ключом командной строки. Выходной текст на
языке toml попадает в файл, путь к которому задан ключом командной строки.
Многострочные комментарии:
%{
Это многострочный
комментарий
%}
Массивы:
#( значение, значение, значение, ... )
Имена:
[A-Z]+
Значения:
• Числа.
• Строки.
• Массивы.
Строки:
@"Это строка"
Объявление константы на этапе трансляции:
36
set имя = значение;
Вычисление константного выражения на этапе трансляции (постфиксная
форма), пример:
?[имя 1 +]
Результатом вычисления константного выражения является значение.
Для константных вычислений определены операции и функции:
1. Сложение.
2. Вычитание.
3. Умножение.
4. Деление.
5. chr().
Все конструкции учебного конфигурационного языка (с учетом их
возможной вложенности) должны быть покрыты тестами. Необходимо показать 2
примера описания конфигураций из разных предметных областей.

## Код программы config_parser.py
```
import re
import toml
import sys

def parse_config(input_file, output_file):
    try:
        with open(input_file, 'r', encoding='utf-8') as f:
            data = f.read()

        # 1. Удаление многострочных комментариев
        data = re.sub(r'%\{.*?%\}', '', data, flags=re.DOTALL)

        config = {}

        # 2. Обработка констант (set NAME = value;)
        for match in re.finditer(r'set\s+([A-Z]+)\s*=\s*(.+?);', data):
            name, value = match.groups()
            config[name] = eval_expression(value.strip())

        # 3. Обработка массивов (#(value, value, ...))
        for match in re.finditer(r'#\((.*?)\)', data):
            values = [eval_expression(v.strip()) for v in match.group(1).split(',')]
            config['array'] = values

        # 4. Обработка строк (@"строка")
        for match in re.finditer(r'@\"(.*?)\"', data):
            config['string'] = match.group(1)

        # 5. Запись результата в файл TOML
        with open(output_file, 'w', encoding='utf-8') as f:
            toml.dump(config, f)
        print(f"Parsing complete. Output saved to {output_file}")
    except Exception as e:
        print(f"Error: {e}")

def eval_expression(expr):
    try:
        # Разрешённые операции и функции: +, -, *, /, chr
        allowed_names = {'chr': chr}
        return eval(expr, {"__builtins__": None}, allowed_names)
    except Exception as e:
        print(f"Invalid expression: {expr}")
        return None

if __name__ == "__main__":
    if len(sys.argv) != 3:
        print("Usage: python3 config_parser.py <input_file> <output_file>")
    else:
        parse_config(sys.argv[1], sys.argv[2])

```

Запуск программы 

``` python3 config_parser.py input.txt output.toml ```

## Входной файл input.txt
```
%{
Это многострочный комментарий
%}

set CONST = 10 + 5;
#( 1, 2, 3, chr(65) )
@"Пример строки"
```

Запускаются из командной строки

``` pytest test_json_to_custom.py ```

Результаты тестирования:
<img width="810" alt="Снимок экрана 2024-12-27 в 9 48 56 AM" src="https://github.com/user-attachments/assets/429b2a11-9bf7-4a9b-b63a-7cb060965fb5" />





