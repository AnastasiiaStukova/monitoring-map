import pandas as pd
import numpy as np

# Read the CSV file with semicolon delimiter
#df = pd.read_csv(r'...\afqk jfa.csv'', delimiter=';', encoding='utf-8')
df = pd.read_csv(r'.....\afqk jfa.csv', delimiter=';', encoding='cp1251')

reference_brands = df.iloc[:, 0].dropna().tolist()

brand_mapping = {}
for brand in reference_brands:
    variations = [
        brand.lower(),
        brand.upper(),
        brand.title(),
        brand.strip(),
        brand.replace(' ', ''),
        brand.replace('-', ' '),
        brand.replace('&', 'and'),
        brand.replace('_', ' '),
        brand.replace('.', ''),
        brand.replace(',', ''),
        brand.replace('  ', ' '),  # двойные пробелы
        brand.replace('/', ' '),
        brand.replace('\\', ' '),
        brand.replace('(', ''),
        brand.replace(')', ''),
        brand.replace('[', ''),
        brand.replace(']', ''),
        brand.replace('"', ''),
        brand.replace('`', ''),
        brand.replace("'", ''),
        f"{brand} ",
        f" {brand}",
        f"{brand}.",
        f"{brand}®",
        f"{brand}™",
        brand.replace('®', ''),
        brand.replace('™', ''),
        brand.replace('©', ''),
        brand.replace('  ', ' ').strip(),  # убираем лишние пробелы
        ' '.join(brand.split()),  # нормализация пробелов
        brand.replace('Corporation', '').strip(),
        brand.replace('Corp.', '').strip(),
        brand.replace('Inc.', '').strip(),
        brand.replace('Limited', '').strip(),
        brand.replace('Ltd.', '').strip(),
        brand.replace('LLC', '').strip(),
        brand.replace('L.L.C.', '').strip(),
        brand.replace('Co.', '').strip(),
        brand.replace('Company', '').strip(),
        brand.replace('International', '').strip(),
        brand.replace('Int.', '').strip(),
        brand.replace('Cosmetics', '').strip(),
        brand.replace('Beauty', '').strip(),
        brand.replace('Professional', '').strip(),
        brand.replace('Prof.', '').strip(),
        brand.replace('®', '').strip(),
        brand.replace('™', '').strip(),
        brand.replace('Laboratories', '').strip(),
        brand.replace('Labs', '').strip(),
        brand.replace('Lab.', '').strip(),
        brand.replace('Group', '').strip(),
        brand.replace('Brands', '').strip(),
        brand.replace('Products', '').strip(),
        brand.replace('By', '').strip(),
        brand.replace('The', '').strip(),
        # Специфичные для парфюмерии
        brand.replace('Parfums', '').strip(),
        brand.replace('Perfumes', '').strip(),
        brand.replace('Fragrances', '').strip(),
        brand.replace('Collection', '').strip(),
        brand.replace('Paris', '').strip(),
        brand.replace('London', '').strip(),
        brand.replace('New York', '').strip(),
        # Обработка специальных символов
        ''.join(c for c in brand if c.isalnum() or c.isspace()),
        # Множественные замены
        brand.replace('  ', ' ').replace('  ', ' ').strip(),
        # Сокращения
        brand.replace('Dr.', 'Doctor').strip(),
        brand.replace('Mr.', 'Mister').strip(),
        brand.replace('Mrs.', 'Misses').strip(),
        brand.replace('Prof.', 'Professor').strip(),
        # Удаление года из названия
        ''.join([i for i in brand if not i.isdigit()]).strip(),
    ]
    
    # Добавляем все вариации в маппинг
    for variation in variations:
        if variation and len(variation) > 1:  # Проверяем, что вариация не пустая и не слишком короткая
            brand_mapping[variation] = brand

def standardize_brand(brand):
    if pd.isna(brand):
        return brand
    brand_str = str(brand).strip()
    brand_lower = brand_str.lower()
    brand_mapping_lower = {k.lower(): v for k, v in brand_mapping.items()}
    if brand_lower in brand_mapping_lower:
        return brand_mapping_lower[brand_lower]
    return brand_str

# Получаем стандартизированные данные
standardized_df = df.apply(lambda x: x.apply(standardize_brand))

# Создаем новый DataFrame для результата
result_df = pd.DataFrame()
result_df['Бренд'] = standardized_df.iloc[:, 0]  # Первая колонка - бренды

# Заполняем колонки конкурентов
competitors = [
   '---', ...
]

for i, competitor in enumerate(competitors, 1):
    result_df[competitor] = ''  # Создаем пустые колонки
    
    # Если бренд найден в соответствующей колонке конкурента, ставим 1
    for index, row in standardized_df.iterrows():
        if competitor.lower() in [col.lower() for col in standardized_df.columns]:
            competitor_col = next(col for col in standardized_df.columns if col.lower() == competitor.lower())
            if not pd.isna(row[competitor_col]) and row[competitor_col].strip() != '':
                result_df.at[index, competitor] = '1'

# Удаляем пустые строки
result_df = result_df.dropna(subset=['Бренд'])

# Заменяем пустые значения на пустые строки
result_df = result_df.fillna('')

# Save the standardized data
result_df.to_csv(r'....standardized_brands.csv', sep=';', index=False, encoding='utf-8')
