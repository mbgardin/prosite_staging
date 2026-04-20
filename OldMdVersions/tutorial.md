# Mastering Regular Expressions for Data Cleaning

![Regex for Data Cleaning](images/regex_banner.png)

[← Back to Home](index.md)

## Introduction

If you have ever spent hours manually fixing cell after cell in Excel, or writing endless `if-else` statements and `.split()` chains in Python to extracting specific information from a text file, you know the pain of messy data. It is often said that data scientists spend 80% of their time cleaning data and only 20% analyzing it. While that statistic might be anecdotal, the struggle is very real.

Enter **Regular Expressions** (often shortened to **Regex**). Regex is a sequence of characters that specifies a search pattern. It is essentially a mini-programming language embedded within other languages (like Python, R, and SQL) designed specifically for text processing.

In this tutorial, we will explore how to use Regex in Python to turn a nightmare dataset into a clean, analysis-ready resource. We will cover the basic syntax, the capabilities of Python's `re` module, and walk through a practical example of cleaning unstructured contact information.

## Why Learn Regex?

You might be wondering, "Can't I just use `.replace()` or string slicing?" For simple cases, absolutely. But Regex shines when you need to match **patterns**, not just specific substrings.

For example, finding "John" in a text is easy. Finding "any word that starts with J, consists of 4 letters, and is followed by a number" is a job for Regex.

Learning Regex will:
1.  **Save you time:** Automate complex text replacements that would take hours manually.
2.  **Reduce code complexity:** Replace 20 lines of string manipulation logic with 1 or 2 lines of Regex.
3.  **Enhance your data wrangling toolkit:** Handle formats like dates, emails, phone numbers, and URLs with ease.

---

## The Building Blocks of Regex

Regex can look intimidating at first—like a cat walked across your keyboard. However, it is built on a few core symbols. Here is a reference table for the most common ones you will encounter in data science:

| Symbol | Name | Description | Example | Matches |
| :--- | :--- | :--- | :--- | :--- |
| `.` | Dot | Matches any single character (except newline) | `a.c` | "abc", "a@c", "a c" |
| `*` | Asterisk | Matches 0 or more of the preceding element | `ab*c` | "ac", "abc", "abbc" |
| `+` | Plus | Matches 1 or more of the preceding element | `ab+c` | "abc", "abbc" (not "ac") |
| `?` | Question Mark | Matches 0 or 1 of the preceding element | `colou?r` | "color", "colour" |
| `\d` | Digit | Matches any digit (0-9) | `\d\d` | "42", "07" |
| `\w` | Word Character | Matches any alphanumeric character (a-z, A-Z, 0-9, _) | `\w+` | "Hello_123" |
| `\s` | Whitespace | Matches any whitespace (space, tab, newline) | `Hello\sWorld` | "Hello World" |
| `^` | Caret | Anchors to the start of the string | `^Hello` | "Hello world" (at start) |
| `$` | Dollar | Anchors to the end of the string | `end$` | "The end" (at end) |
| `[]` | Character Set | Matches any one character inside the brackets | `[aeiou]` | "a", "e" |
| `()` | Group | Groups patterns together (captures the match) | `(abc)+` | "abc", "abcabc" |

Understanding these building blocks is key to constructing powerful patterns.

---

## Using Regex in Python

Python has a built-in module called `re` for working with regular expressions. Let's look at the three most useful functions for data cleaning: `re.search()`, `re.findall()`, and `re.sub()`.

First, you need to import the library:

```python
import re
```

### 1. Finding a Match: `re.search()`

`re.search()` scans the string and returns the **first** location where the pattern produces a match. It returns a "Match object" if successful, or `None` if not.

```python
text = "The price is $45.99 for the item."
# Pattern: \$ matches a literal dollar sign, \d+ matches digits, \. matches literal dot
pattern = r"\$\d+\.\d+"

match = re.search(pattern, text)

if match:
    print(f"Found match: {match.group()}")
else:
    print("No match found.")

# Output: Found match: $45.99
```

*Note: We use `r"string"` to denote a "raw string" in Python, which handles backslashes correctly for regex.*

### 2. Finding All Matches: `re.findall()`

Often you want to extract *all* instances of a pattern, not just the first one. `re.findall()` returns a list of strings.

```python
text = "Contact emails: support@example.com, sales@example.org, hr@company.net"
# Pattern: \w+ matches username, @ matches @, [\w\.]+ matches domain
email_pattern = r"[\w\.-]+@[\w\.-]+"

emails = re.findall(email_pattern, text)
print(emails)

# Output: ['support@example.com', 'sales@example.org', 'hr@company.net']
```

### 3. Replacing Text: `re.sub()`

This is the search-and-replace function. It replaces occurrences of the pattern with a replacement string.

```python
date_string = "Report generated on 2023-10-27"
# Change YYYY-MM-DD to DD/MM/YYYY
# We use groups () to capture parts of the date
pattern = r"(\d{4})-(\d{2})-(\d{2})"
replacement = r"\3/\2/\1" # \1 refers to the first group, etc.

new_string = re.sub(pattern, replacement, date_string)
print(new_string)

# Output: Report generated on 27/10/2023
```

---

## Practical Example: Cleaning Messy Phone Numbers

Let's apply this to a realistic scenario. Imagine you scraped a website or received a dataset from a client with a "Phone Number" column that is completely inconsistent.

Here is our raw data:

```python
phone_numbers = [
    "Call me at 555-0199",
    "(555) 123-4567",
    "555.987.6543",
    "Phone: 555 123 4567",
    "No number here",
    "+1-555-555-5555"
]
```

We want to extract just the numbers and format them as `(XXX) XXX-XXXX`.

### Step 1: Analyze the Pattern

We can see a few common structures:
- Optional area code (3 digits)
- Separators like dash `-`, dot `.`, space ` `, or parens `()`
- A 3-digit prefix
- A 4-digit line number

### Step 2: Build the Regex

A naive approach might be to look for "digit-digit-digit" patterns. But let's build a flexible one.

We want to find precisely 10 digits (ignoring the country code `+1` for simplicity in this tutorial, or we could strip it first).

Actually, the easiest cleaning pattern in Python is to **remove everything that is NOT a digit**, and then format the remaining digits.

```python
cleaned_numbers = []

for entry in phone_numbers:
    # re.sub with [^\d] means "replace anything that is NOT a digit" with empty string
    digits_only = re.sub(r"[^\d]", "", entry)
    
    # Check if we have a valid length (e.g., 10 digits for US numbers)
    # Sometimes we might get '1' from the country code at the start, let's handle that simple case
    if len(digits_only) == 11 and digits_only.startswith('1'):
         digits_only = digits_only[1:]
    
    if len(digits_only) == 10:
        formatted = f"({digits_only[:3]}) {digits_only[3:6]}-{digits_only[6:]}"
        cleaned_numbers.append(formatted)
    else:
        cleaned_numbers.append("Invalid Format")

print(cleaned_numbers)
```

**Output:**
```
['Invalid Format', '(555) 123-4567', '(555) 987-6543', '(555) 123-4567', 'Invalid Format', '(555) 555-5555']
```

Wait, look at the first entry `"Call me at 555-0199"`. It only had 7 digits. Our logic correctly marked it as "Invalid Format" because it's missing an area code.

### Step 3: Advanced Extraction

What if we wanted to *extract* the number from the text first, rather than just cleaning the whole string?

```python
text = "Reach us at (555) 123-4567 for support."
# Pattern explanation:
# \(?     : Optional opening parenthesis
# (\d{3}) : Capture Group 1: Area code (3 digits)
# \)?     : Optional closing parenthesis
# [\s\.-]?: Optional separator (space, dot, or dash)
# (\d{3}) : Capture Group 2: Prefix (3 digits)
# [\s\.-]?: Optional separator
# (\d{4}) : Capture Group 3: Line number (4 digits)
pattern = r"\(?(\d{3})\)?[\s\.-]?(\d{3})[\s\.-]?(\d{4})"

match = re.search(pattern, text)
if match:
    area_code, prefix, line = match.groups()
    print(f"Extracted: {area_code}-{prefix}-{line}")

# Output: Extracted: 555-123-4567
```

This approach is much more precise and avoids accidentally picking up stray numbers (like a year "2023" or an ID "12345") that don't look like phone numbers.

---

## Conclusion and Next Steps

Regular expressions are a fundamental skill for any data scientist. While they can be tricky to write initially, they provide a level of precision and power that standard string methods cannot match.

In this tutorial, we covered:
- **Core Regex symbols** like `\d`, `*`, `+`, and character sets `[]`.
- **Python usage** with `re.search`, `re.findall`, and `re.sub`.
- **A cleaning workflow** to normalize inconsistent data.

### Call to Action

Now it's your turn!
1.  **Review your data:** Look at the datasets you are using for your class projects. Are there messy text columns?
2.  **Practice:** Try to write a regex to extract all the hashtags from a sample tweet string (Hint: starts with `#`).
3.  **Explore:** Check out [Regex101](https://regex101.com/), an amazing interactive tool to test and debug your patterns in real-time.

Mastering regex takes practice, but once you "speak the language," you'll see patterns everywhere. Happy coding!
