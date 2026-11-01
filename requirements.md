# Book Price Watch

## Goal

A small bookshop compares its prices with an online competitor every week. The owner wants the books of a few categories in one Excel file, sorted by price, with the cheap ones marked, so she can decide which titles to discount.

## Steps

1. Open the competitor's shop: https://books.toscrape.com/
2. For each category in the settings, open that category from the list on the left of the home page.
3. Read every book in the category: its title, price, star rating, and whether it is in stock. A category can have more than one page; follow "next" until the last page, so no book is left out.
4. Keep the price as a number: the page shows "£51.77", the file needs 51.77 without the currency sign.
5. Turn the star rating into a number from 1 to 5 (the page shows it as a word such as "Three").
6. Mark a book as "cheap" when its price is below the price limit in the settings.
7. If a category in the settings does not exist on the site, stop and say which one, instead of writing a file without it.
8. Save all books to one Excel file, sorted by category and then by price, from low to high.
9. Write one line of summary: how many books were read in each category, and how many are cheap.
10. Close the browser at the end, also when something went wrong.

## Settings that must change without editing the flow

- The address of the shop
- The categories to read (for now: Travel, Poetry, Humor)
- The price limit for "cheap", in GBP (20 for now)
- Whether the browser window is shown while the flow runs

## Secrets

- none

## Personal or confidential data

- none (the shop and its books are a public practice site)

## Output

All in `output/`:

- `books.xlsx`: one sheet named "Books", one row per book, with the columns category, title, price, rating, in_stock, and cheap
- `summary.txt`: for example "Travel: 11 books, 3 cheap. Poetry: 19 books, 6 cheap. Humor: 10 books, 2 cheap."

## Errors and exceptions

- If the site does not respond, try again twice, then stop.
- If a category in the settings is not on the site, stop and name it.
- If a category shows no books, stop and name the category.
- If a book has no title or no price, stop and say which book, instead of saving a wrong row.

## Schedule

By hand while testing. Later: every Monday at 07:00, on a worker.
