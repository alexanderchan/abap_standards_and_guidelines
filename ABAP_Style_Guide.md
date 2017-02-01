
# Convergent IS ABAP Style Guide

A mostly reasonable approach to ABAP

# General Guidelines

- Client standards should be followed over this document
- Keep code consistent, readable, and understandable 

## Table of Contents

1. [Naming](#naming)
1. [Variable Declaration](#variable-declaration)

## Naming

- Use meaningful long names
- Variables should be named with the following prefixes:

| prefix | Usage                  | Example / Notes                                        |
|:------:|------------------------|--------------------------------------------------------|
|    g   | Global (or static)     | `gv_plvar`                                             |
|    c   | Global Constant        | `c_status_active` - *note that gc_ is also acceptable* |
|    l   | Local variables        | `ls_mara`                                              |
|    m   | Class member variables | Member variables are kept with the instance            |



## Variable Declaration

- All variables should be declared as close as possible to their usage
