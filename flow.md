# Book Price Watch
> Description: Collect book details by category from an online bookshop, mark cheap books, and save results to Excel and a summary file.
> Version: 1.0.0
> Requires: kinenix>=1.0.0rc2

## Variables
- `all_books`: []
- `summary_parts`: []

## Steps

### 1. Open Competitor Shop (`web.open`)
- **url:** ${config.shop_url}
- **headless:** ${config.headless}
- **max_retries:** 2
- **retry_interval:** 5.0

### 2. Loop Through Categories (`logic.loop`)
- **items:** ${config.categories}
- **item_var:** `category`
- **item_label:** ${category}
- **Sub-steps:**
  - Return To Home Page (`web.goto`):
    - **url:** ${config.shop_url}
    - **max_retries:** 2
    - **retry_interval:** 2.0
  - Check Category Exists (`web.is_visible`):
    - **selector:** `xpath=//div[contains(@class, 'side_categories')]//a[normalize-space()='${category}']`
    - **timeout:** 3000
    - **output_var:** `category_exists`
  - Stop If Category Not Found (`flow.fail`):
    - **condition:** `${category_exists} == false`
    - **message:** Category '${category}' was not found on the site.
  - Open Category (`web.click`):
    - **selector:** `xpath=//div[contains(@class, 'side_categories')]//a[normalize-space()='${category}']`
    - **max_retries:** 2
    - **retry_interval:** 2.0
  - Read Books In Category (`web.get_list`):
    - **selector:** `article.product_pod`
    - **fields:** {"title": "h3 a@title", "price": "p.price_color", "rating_text": "p.star-rating@class", "stock_text": "p.instock"}
    - **numeric_fields:** ["price"]
    - **required_fields:** ["title", "price"]
    - **next_selector:** `li.next a`
    - **output_var:** `category_books`
  - Stop If Category Has No Books (`flow.fail`):
    - **condition:** `${len(category_books)} == 0`
    - **message:** Category '${category}' shows no books.
  - Reset Cheap Count (`logic.set_variable`):
    - **name:** cheap_count
    - **value:** 0
  - Process Each Book (`logic.loop`):
    - **items:** ${category_books}
    - **item_var:** `book`
    - **item_label:** ${book.title}
    - **Sub-steps:**
      - Reset Book Rating (`logic.set_variable`):
        - **name:** rating
        - **value:** 0
      - Set Rating One (`logic.set_variable`):
        - **condition:** `${book.rating_text} contains One`
        - **name:** rating
        - **value:** 1
      - Set Rating Two (`logic.set_variable`):
        - **condition:** `${book.rating_text} contains Two`
        - **name:** rating
        - **value:** 2
      - Set Rating Three (`logic.set_variable`):
        - **condition:** `${book.rating_text} contains Three`
        - **name:** rating
        - **value:** 3
      - Set Rating Four (`logic.set_variable`):
        - **condition:** `${book.rating_text} contains Four`
        - **name:** rating
        - **value:** 4
      - Set Rating Five (`logic.set_variable`):
        - **condition:** `${book.rating_text} contains Five`
        - **name:** rating
        - **value:** 5
      - Set In Stock True (`logic.set_variable`):
        - **condition:** `${book.stock_text} contains 'In stock'`
        - **name:** in_stock
        - **value:** true
      - Set In Stock False (`logic.set_variable`):
        - **condition:** `${book.stock_text} not contains 'In stock'`
        - **name:** in_stock
        - **value:** false
      - Set Cheap True (`logic.set_variable`):
        - **condition:** `${book.price} < ${config.cheap_price_limit}`
        - **name:** cheap
        - **value:** true
      - Set Cheap False (`logic.set_variable`):
        - **condition:** `${book.price} >= ${config.cheap_price_limit}`
        - **name:** cheap
        - **value:** false
      - Count Cheap Book (`logic.set_variable`):
        - **condition:** `${book.price} < ${config.cheap_price_limit}`
        - **name:** cheap_count
        - **value:** ${cheap_count + 1}
      - Collect Book (`logic.append`):
        - **target:** all_books
        - **item:** {"category": "${category}", "title": "${book.title}", "price": "${book.price}", "rating": "${rating}", "in_stock": "${in_stock}", "cheap": "${cheap}"}
  - Record Category Summary (`logic.append`):
    - **target:** summary_parts
    - **item:** ${category}: ${len(category_books)} books, ${cheap_count} cheap.

### 3. Save Books To Excel (`excel.write`)
- **file_path:** output/books.xlsx
- **data:** ${all_books}
- **sheet_name:** Books
- **columns:** ["category", "title", "price", "rating", "in_stock", "cheap"]

### 4. Save Summary (`file.write_text`)
- **path:** output/summary.txt
- **content:** ${join(summary_parts, " ")}

## Finally

### 1. Close Browser (`web.close`)
- **on_error:** continue
