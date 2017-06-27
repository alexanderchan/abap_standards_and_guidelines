
# Convergent IS ABAP Style Guide

A mostly reasonable and pretty opinionated approach to ABAP.

## General Guidelines

- Client standards should be followed over this document
- Keep code consistent, readable, and understandable

## Table of Contents

1. [Naming](#naming)
1. [Variable Declaration](#variable-declaration)

## Naming

- Use meaningful long names
- Variables should be named with the following prefixes:

| prefix | Usage                  | Example / Notes                                                  |
|:------:|------------------------|--------------------------------------------------------          |
|    g   | Global (or static)     | `gv_plvar`                                                       |
|    c   | Global Constant        | `c_status_active` <br/> *note that gc_ is also acceptable*       |
|    l   | Local variables        | `ls_mara`                                                        |
|    m   | Class member variables | `mt_cached_data` Member variables are kept with the instance     |

### Second character

| character | Usage                  | Example / Notes                                                  |
|:------:|------------------------|--------------------------------------------------------          |
|    t   | table     | `lt_plvar`                                                       |
|    v   | single variable        | `lv_position` |
|    s   | Structure/work area       | `ls_mara` a table structure                                                       |


## Performance

- Go for O constant wherever possible, everything else will slow down your code.

![LogN](https://i.stack.imgur.com/8nXvk.jpg)

- Always read from hashed tables and assign the results to a field symbol.  Avoid both standard and sorted tables for reads
  - The performance for a read is constant
  - This can improve the performance of almost any code processing large quantities of data

```abap
" Slow
data: lv_position type pa0001-plans.
loop at lt_people into ls_people.  
  select single PLANS into lv_position from pa0001
    where pernr = ls_people-pernr
      and begda <= lv_key_date and endda >= lv_key_date.
  if sy-subrc = 0.
    ls_people-position = lv_position.
  endif.
  modify lt_people from ls_position.
endloop.

" Good
" Use the following pattern to gather all of the data up front
" then 'join' it back together in a loop and read
data: lt_pernr_position_h type hashed table of ty_s_pernr_position with unique key pernr,
      lt_pernr_position type standard table of ty_s_pernr_position.

if lt_people is not initial.
  select pernr plans from pa0001 into table lt_pernr_position
  for all entries in lt_people
  where pernr = lt_people-pernr
    and begda <= lv_key_date and endda >= lv_key_date.

  " If one is not sure that the results will be unique, read into a standard table
  " and filter/sort/delete adjacent duplicates
  " and then move from the standard table into the hashed table
  " don't forget to free the resources from the standard table
  " here we know that there is only one result, but as an example
  sort lt_pernr_position by pernr.
  delete adjacent duplicates from lt_pernr_position.
  lt_pernr_position_h = lt_pernr_position.

  loop at lt_people assigning field-symbol(<ls_people>).
    read table lt_pernr_position_h
          assigning field-symbol(<ls_pernr_position>)
          with table key pernr = <ls_people>-pernr.
    if sy-subrc = 0.
      <ls_people>-position = <ls_pernr_position>-plans.
    endif.
  endloop.

endif. " End of for all entries check.
```


- Following the above, never read using binary search or use a sorted table unless there is a good reason. Reading from a sorted table with a unique key will give the same result as a hashed table but slower (except for very small numbers).  Using a hashed table forces one to think about the correct key in advance and will allow the program to scale.
  - The performance for a sorted read is oLog(N) - proportional to the number of records read


- Select using indexes
- Group as much of the selects and function reads outside of the main loop by using for all entries and then reading afterwards with
- Use field symbols for reading and looping
- Read internal tables with the addition `WITH TABLE KEY`


## OpenSQL

- Always check sy-subrc and do something with the result
- Always check the table being used in a for all entries has values

```abap
" Bad
select matnr from mara for all entries in lt_materials ...

" Good
if lt_materials is not initial
  select matnr from mara for all entries in lt_materials ...
endif.

" Why?
" If the for all entries internal table is not checked, ALL data will be read
```
### Joins

- Don't rename the tables that are being selected from

```abap
" Bad
select a~matnr b~werks
  into table lt_matnr_werks
  from mara as a join marc as b
    on a~matnr = b~matnr
    where ...

" Good
select mara~matnr marc~werks
  into table lt_matnr_werks
  from mara join marc
    on mara~matnr = marc~matnr
    where ...

" Why?
" Renaming variables saves typing but leads to other developers trying to
" reverse engineer which variable is which and to keep a mapping of what a or b or c equals
" Please keep it simple and just use the table
```

- Keep multiple joins to a minimum
  * Why?

    More than 3 joins can be hard to debug and validating indexes can get complex.  Keep joins short and small and join using read tables techniques.
## Variable Declaration

- All variables should be declared as close as possible to their usage

```ABAP
" Harder to refactor:
data: ls_stuff type x,
      lt_more type y.

ls_stuff = ...

" Locate things closer to the code, it will allow
" for 'copy paste' refactoring rather than 'copy, try to extract the variables
" from the header, hope that you have everything' and try again refactoring


```
- If we know the netweaver stack is high enough, inline declarations (except for in OpenSQL) are preferred for the same reasons as declare as close as possible.  In addition, data types don't need to be known and updated whenever variables change

```ABAP
" Old style
field-symbols: <ls_mara> type mara.
read table lt_mara assigning <ls_mara>.

data: ls_person_details type zcl_class=>ty_person_details.
call method zcl_class=>get_person_details(
  importing
    es_person_details = ls_person_details
  )

" New style - shorter and less brittle
read table lt_mara assigning field-symbol(<ls_mara>).
zcl_class=>get_person_details(
  importing
  es_person_details = data(ls_person_details)
  )

```




```

## Useful tools

[Editing Markdown Tables](http://www.tablesgenerator.com/markdown_tables)
