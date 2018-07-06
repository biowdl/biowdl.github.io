---
layout: default
title: BioWDL Style Guidelines
---

## 1. Indentation ##
What should be indented:
1. Anything within a set of braces `{}`.
2. Inputs following `input:` in a call block.
3. Continuations of expressions which did no fit on a single line, see section
   4.

Indentations should be 4 space characters (` `) per level of indentation.
Closing braces should use the same level as indentation as their opening
braces.

> Do not use a tab character.

### Examples ###

**yay:**

```
workflow example {
    call SomeTask as doStuff {
        input:
            number = 1,
            letter = "a"
    }
}
```

**nay:**
- Contents of workflow block indented with 2 spaces, rather than 4.
- Inputs not indented.

```
workflow example {
  call SomeTask as doStuff {
      input:
      number = 1,
      letter = "a"
  }
}
```

## 2. Blank lines ##
Blank lines should be used to separate different parts of a workflow:
- Different blocks (code surrounded by `{}`) should be separated by a single
  blank line.
- Different groupings of inputs (either in call blocks or task blocks,
  may also be separated by a single blank line.
- Inputs should always be followed by a single blank line, unless there is
  nothing beneath them besides closing braces (`}`).
- Between the closing braces of a parent and child block, no blank lines should
  be placed.

### Examples ###

**yay:**

```
task echo {
    String message

    String? outputPath # Optional input(s) separated from mandatory input(s)

    command {
        echo ${message} ${"> " + outputPath}
    }

    output {
        File? outputFile = outputPath
    }
}
```

**nay:**
- Double blank line between inputs and `command` block.
- No blank line between `command` and `output` block.
- Pointless blank line between closing braces.

```
task echo {
    String message

    String? outputPath


    command {
        echo ${message} ${"> " + outputPath}
    }
    output {
        File? outputFile = outputPath
    }

}
```

## 3. Expression spacing ##
Expressions should be spaced out to improve readability.

### Calculations and evaluations ###
Spaces should be added between values and operators in calculations and
evaluations. If multiple evaluations or calculations occur within a single
expression, they may be grouped without placing spaces, with spaces being
placed between the groups instead. It is advisable to base the groups on the
order or operations.  
In the case of groupings, opening brackets (`(`) should always be preceded by a
space and closing brackets (`)`) should always be followed by one. There should
not be a space between the brackets and their first or last value.  
In the case of function calls, there should *not* be a space between the
function's name and the opening bracket of it's parameters, but besides that
the same rules apply as with groupings, as far as the brackets are concerned.
The commas separating the parameters should be followed by a space.

There are a number of operators which should always be surrounded by spaces:
- Less than: `<`
- Less then or equal: `<=`
- Greater than: `>`
- Greater than or equal: `>=`
- Equal: `==`
- Not Equal: `!=`
- Logical AND: `&&`
- Logical OR: `||`
- Assignment: `=`

There are also a number of operators which should always be preceded by a
space, but not followed by one (even when they precede a bracket). If these
operators apply to the result of an expression. The expression should be
surrounded by brackets.
- Logical NOT: `!`
- Unary plus: `+`
- Unary negation: `-`

### Examples ###

**yay:**
- `1 + 1`
- `1 + 1/2`
- `(1 + 1) / 2 `
- `!(a == -1)`

**nay:**
- `1+1`
- `1 + 1 / 2`
- `( 1+1 ) / 2` or `(1 + 1)/2`
- `!(a==-1)` or `! (a == -1)` or `!a == -1`

## 4. Line length and line breaks ##
Lines should be at most 100 characters long. If a line exceeds this, it should
be broken up into multiple lines. The following lines should be indented as
described in section 1.

### Line breaks ###
When using line breaks to adhere to the line length limit, they should occur
on logical places. These include:
- Following a comma.
- Before the `then` or `else` in an `if-then-else` expression.
- Following an opening bracket (`(`).
- Following an operators which would otherwise be followed by a space (see
  section 3), as a last resort.

In addition there are some places where there should always be a line break.
These include:
- Between inputs in a call block.
- Following `inputs:` in a call block.
- Following opening braces (`{`).
- Before closing braces (`}`).

### Examples ###

**yay:**

```
Int value = if defined(aVariableThatHasAWayTooLongName)
    then aVariableThatHasAWayTooLongName else 10

# or

Int value = if defined(aVariableThatHasAWayTooLongName)
    then aVariableThatHasAWayTooLongName
    else 10

# ----

call SomeTask as doStuff {
    input:
        number = 1,
        letter = "a"
}

```

**nay:**

```
# too long
Int value = if defined(aVariableThatHasAWayTooLongName) then aVariableThatHasAWayTooLongName else 10

# or wrong line break
Int value = if defined(aVariableThatHasAWayTooLongName) then
    aVariableThatHasAWayTooLongName else 10

# ----

# multiple inputs on one line
call SomeTask as doStuff {
    input:
        number = 1, letter = "a"
}

# missing line breaks
call SomeTask as doStuff {input: number = 1, letter = "a"}
```

## 5. Naming conventions ##
All imports and calls should be named using the `as` syntax. The names for the
various different types of objects should be formatted as listed below:
- Tasks: UpperCamelCase
- Workflows: UpperCamelCase
- Calls: lowerCamelCase
- Variables: lowerCamelCase
- Inputs: see the notes below
- Imports: lowerCamelCase

### Input names ###
Input names should mimic the (long form) versions of the options they represent
as much as possible. This entails that if there is a version of the option
which is longer than 1 character, the associated input nam should be the same
as this long form option. Preferably the name would be reformatted to
lowerCamelCase, though some exceptions may exist, such as cases in which the
name contains capitalized abbreviation. A word following this abbreviation may
be uncapitalized.
In any other situation use lowerCamelCase.
When naming an input which represents a file path, which is not associated with
a command option for which a long form exists, the name of this input should
end with the word `Path`. Note that this is only the case if the input is of
the `String` or `String?` type.

### Avoiding name conflicts ###
`Task` or `Flow` may be added at the end of imported tasks or workflow in order
to avoid conflicts in names between calls and imports, if it can otherwise not
be avoided. Do not change the name of the call.

### Examples ###

**yay:**

```
task DoStuff {
    File inputFile
    String outputPath

    Int maxRAM

    command {
        someScript \
        -i ${inputFile} \
        --maxRAM ${maxRAM} \
        -o outputPath
    }

    output {
        File output = outputPath
    }
}
```

**nay:**
- task name is not UpperCamelCase
- `input_file` does not use lowerCamelCase
- `OutputPath` does not use lowerCamelCase
- `ram` does not match the option it is represents

```
task doStuff {
    File input_file
    String OutputPath

    Int ram

    command {
        someScript \
        -i ${input} \
        --maxRAM ${maxRAM} \
        -o outputPath
    }

    output {
        File output = outputPath
    }
}
```
## 6. Workflow structure ##
In general tasks and workflows should be kept in separate files. Only if a
task is small and specific to a certain workflow should it be in the same file
as the workflow.
Tasks relating to the same tool or toolkit should be in the same file.

If a file contains multiple tasks, they should be ordered alphabetically.
Calls in a workflow should be placed in order of execution.

## 7. Command block ##
1. Each option in a bash command should be on a new line. Ending previous lines
   with a backslash (`\`). Some grouping of options on a single line may be
   acceptable. For example, various java memory settings may be set on the same
   line, as long as the line does not exceed the line length limit as described
   in section 4.
2. All bash commands should start with `set -eo pipefail`.
3. All commands should have a `${preCommand}` following point 2 and before the
   actual command. Preferably *directly* before the actual command. The
   preCommand should be settable as an inputs.

### Examples ###

**yay:**

```
task doStuff {
    File inputFile
    String outputPath
    Int maxRAM

    String? preCommand

    command {
        set -eo pipefail
        ${preCommand}
        someScript \
        -i ${inputFile} \
        --maxRAM ${maxRAM} \
        -o outputPath
    }

    output {
        File output = outputPath
    }
}
```

**nay:**
- all option are on the same line
- `set -eo pipefail` is absent
- `preCommand` is absent

```
task doStuff {
    File inputFile
    String outputPath

    Int maxRAM

    command {
        someScript -i ${inputFile} --maxRAM ${maxRAM} -o outputPath
    }

    output {
        File output = outputPath
    }
}
```

## 8. Imports ##
Imports should be placed in alphabetical order at the top of the file. All
imports must be named using the `as` syntax.

## 9. Style enforcement ##
For now there is no automated way of styling or check the style of WDL files of
according to these guidelines. It is up to the authors and reviewers to ensure
the guidelines are adhered to.
