# Book Price Watch
> Description: Read every book in the configured categories from Books to Scrape, mark cheap titles below the price limit, save them to an Excel file sorted by category and price, and write a summary file.
> Version: 1.0.0
> Requires: kinenix>=1.0.0rc1

## Variables
- `all_books`: []
- `summary_parts`: []
- `stars_map`: {"One": 1, "Two": 2, "Three": 3, "Four": 4, "Five": 5}

## Steps

### 1. Open Competitor Shop (`web.open`)
- **url:** `${config.shop_url}`
- **headless:** `${config.headless}`
- **timeout:** 30000
- **max_retries:** 2
- **retry_interval:** 3

### 2. Read Each Category (`logic.loop`)
- **items:** `${config.categories}`
- **item_var:** `category`
- **item_label:** `${category}`
- **Sub-steps:**
  - Return To Shop Home (`web.goto`):
    - **condition:** `${category_index} > 0`
    - **url:** `${config.shop_url}`
    - **timeout:** 30000
    - **max_retries:** 2
    - **retry_interval:** 3
  - Check If Category Exists On Site (`web.is_visible`):
    - **selector:** `.side_categories a:text-is('${category}')`
    - **timeout:** 3000
    - **output_var:** `category_exists`
  - Stop When Category Is Missing (`flow.fail`):
    - **condition:** `${category_exists} == false`
    - **message:** Category '${category}' does not exist on the site.
  - Open Category Page (`web.click`):
    - **selector:** `.side_categories a:text-is('${category}')`
    - **timeout:** 15000
    - **max_retries:** 2
    - **retry_interval:** 2
  - Check If Category Has Books (`web.is_visible`):
    - **selector:** `article.product_pod`
    - **timeout:** 3000
    - **output_var:** `has_books`
  - Stop When Category Shows No Books (`flow.fail`):
    - **condition:** `${has_books} == false`
    - **message:** Category '${category}' shows no books.
  - Read All Books In Category (`web.get_list`):
    - **selector:** `article.product_pod`
    - **fields:** {"title": "h3 a@title", "price_text": "p.price_color", "rating_class": "p.star-rating@class", "availability": "p.availability"}
    - **next_selector:** `li.next a`
    - **max_pages:** 50
    - **timeout:** 30000
    - **output_var:** `raw_books`
    - **max_retries:** 2
    - **retry_interval:** 3
  - Reset Category Books (`logic.set_variable`):
    - **name:** `category_books`
    - **value:** []
  - Reset Cheap Count (`logic.set_variable`):
    - **name:** `cheap_count`
    - **value:** 0
  - Format And Check Each Book (`logic.loop`):
    - **items:** `${raw_books}`
    - **item_var:** `book`
    - **item_label:** `${default(book.title, 'Untitled book')}`
    - **Sub-steps:**
      - Stop When Book Has No Title (`flow.fail`):
        - **condition:** `${book.title} is empty`
        - **message:** Book #${book_index + 1} in category '${category}' has no title.
      - Stop When Book Has No Price (`flow.fail`):
        - **condition:** `${book.price_text} is empty or ${number(book.price_text)} is empty`
        - **message:** Book '${book.title}' in category '${category}' has no price.
      - Set Numeric Price (`logic.set_variable`):
        - **name:** `price_num`
        - **value:** `${number(book.price_text)}`
      - Set Star Rating Number (`logic.set_variable`):
        - **name:** `rating_num`
        - **value:** `${stars_map[part(book.rating_class, ' ', -1)]}`
      - Set Stock Status (`logic.if`):
        - **condition:** `${book.availability} contains 'In stock'`
        - **Sub-steps:**
          - Mark In Stock (`logic.set_variable`):
            - **name:** `in_stock_val`
            - **value:** true
        - **Else-steps:**
          - Mark Out Of Stock (`logic.set_variable`):
            - **name:** `in_stock_val`
            - **value:** false
      - Set Cheap Mark And Count (`logic.if`):
        - **condition:** `${price_num} < ${config.cheap_limit}`
        - **Sub-steps:**
          - Mark As Cheap (`logic.set_variable`):
            - **name:** `cheap_val`
            - **value:** true
          - Increment Cheap Count (`logic.set_variable`):
            - **name:** `cheap_count`
            - **value:** `${cheap_count + 1}`
        - **Else-steps:**
          - Mark As Not Cheap (`logic.set_variable`):
            - **name:** `cheap_val`
            - **value:** false
      - Add Book To Category List (`logic.append`):
        - **target:** `category_books`
        - **item:** {"category": "${category}", "title": "${book.title}", "price": "${price_num}", "rating": "${rating_num}", "in_stock": "${in_stock_val}", "cheap": "${cheap_val}"}
  - Sort Category Books By Price Low To High (`logic.loop`):
    - **items:** `${category_books}`
    - **item_var:** `sort_pass`
    - **item_label:** `${sort_pass.title}`
    - **Sub-steps:**
      - Start With First Remaining Book (`logic.set_variable`):
        - **name:** `min_book`
        - **value:** `${category_books[0]}`
      - Start With First Index (`logic.set_variable`):
        - **name:** `min_idx`
        - **value:** 0
      - Find Cheapest Remaining Book (`logic.loop`):
        - **items:** `${category_books}`
        - **item_var:** `cand`
        - **item_label:** `${cand.title}`
        - **Sub-steps:**
          - Pick Cheaper Book (`logic.if`):
            - **condition:** `${cand.price} < ${min_book.price}`
            - **Sub-steps:**
              - Update Cheapest Book (`logic.set_variable`):
                - **name:** `min_book`
                - **value:** `${cand}`
              - Update Cheapest Index (`logic.set_variable`):
                - **name:** `min_idx`
                - **value:** `${cand_index}`
      - Add Cheapest Book To All Books (`logic.append`):
        - **target:** `all_books`
        - **item:** `${min_book}`
      - Reset Remaining Books (`logic.set_variable`):
        - **name:** `remaining_books`
        - **value:** []
      - Keep Remaining Books (`logic.loop`):
        - **items:** `${category_books}`
        - **item_var:** `rem`
        - **item_label:** `${rem.title}`
        - **Sub-steps:**
          - Keep Book If Not Picked (`logic.append`):
            - **condition:** `${rem_index} != ${min_idx}`
            - **target:** `remaining_books`
            - **item:** `${rem}`
      - Update Category Books (`logic.set_variable`):
        - **name:** `category_books`
        - **value:** `${remaining_books}`
  - Add Category Summary (`logic.append`):
    - **target:** `summary_parts`
    - **item:** `${category}: ${len(raw_books)} books, ${cheap_count} cheap.`

### 3. Save All Books To Excel (`excel.write`)
- **file_path:** `output/books.xlsx`
- **sheet_name:** `Books`
- **data:** `${all_books}`
- **columns:** ["category", "title", "price", "rating", "in_stock", "cheap"]
- **max_retries:** 2
- **retry_interval:** 2

### 4. Write Summary File (`file.write_text`)
- **path:** `output/summary.txt`
- **content:** `${join(summary_parts, ' ')}`
- **max_retries:** 2
- **retry_interval:** 2

## Finally

### 1. Close Browser (`web.close`)
- **on_error:** continue
