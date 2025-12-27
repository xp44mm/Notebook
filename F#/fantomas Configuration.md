Fantomas 配置项中文说明（含取值范围）：

---

### **[Configuration]**
- **`version`**  
  当前 Fantomas 版本号：`7.0.3+`
- 实验性功能（Experimental）可能在未来版本中调整。

---

### **[Auxiliary settings]**
1. **`indent_size`**  
   缩进空格数（1-10，默认4）

2. **`max_line_length`**  
   最大行宽（≥60，默认120）

3. **`end_of_line`**  
   换行符类型：`lf`（`\n`）或 `crlf`（`\r\n`），默认系统决定

4. **`insert_final_newline`**  
   文件末尾添加换行（true/false，默认true）

---

### **[Formatting settings]**
5. **`fsharp_space_before_parameter`**  
   函数定义名与首参数括号间空格（true/false，默认true）

6. **`fsharp_space_before_lowercase_invocation`**  
   小写函数调用与括号间空格（true/false，默认true）

7. **`fsharp_space_before_uppercase_invocation`**  
   大写函数调用与括号间空格（true/false，默认false）

8. **`fsharp_space_before_class_constructor`**  
   类名与构造括号间空格（true/false，默认false）

9. **`fsharp_space_before_member`**  
   成员名与参数括号间空格（true/false，默认false）

10. **`fsharp_space_before_colon`**  
    类型标注冒号前空格（true/false，默认false）

11. **`fsharp_space_after_comma`**  
    元组逗号后空格（true/false，默认true）

12. **`fsharp_space_before_semicolon`**  
    分号前空格（true/false，默认false）

13. **`fsharp_space_after_semicolon`**  
    分号后空格（true/false，默认true）

14. **`fsharp_space_around_delimiter`**  
    集合符号（`[`/`{`等）周围空格（true/false，默认true）

---

### **[Maximum width constraints]**
15. **`fsharp_max_if_then_short_width`**  
    无else的if表达式单行最大宽度（≥0，默认0=强制换行）

16. **`fsharp_max_if_then_else_short_width`**  
    带else的if表达式单行最大宽度（≥0，默认60）

17. **`fsharp_max_infix_operator_expression`**  
    中缀表达式单行最大宽度（≥0，默认80）

18. **`fsharp_max_record_width`**  
    记录类型单行最大宽度（≥1，默认40）

19. **`fsharp_max_record_number_of_items`**  
    记录类型单行最大字段数（≥1，默认1）

20. **`fsharp_record_multiline_formatter`**  
    记录换行依据：`character_width`（按宽度）或 `number_of_items`（按字段数）

21. **`fsharp_max_array_or_list_width`**  
    数组/列表单行最大宽度（≥1，默认80）

22. **`fsharp_max_array_or_list_number_of_items`**  
    数组/列表单行最大元素数（≥1，默认1）

23. **`fsharp_array_or_list_multiline_formatter`**  
    数组/列表换行依据：`character_width` 或 `number_of_items`

24. **`fsharp_max_value_binding_width`**  
    let绑定单行最大宽度（≥1，默认80）

25. **`fsharp_max_function_binding_width`**  
    函数定义单行最大宽度（≥1，默认40）

---

### **[Multiline formatting]**
26. **`fsharp_multiline_bracket_style`**  
    多行括号风格：`cramped`（默认）、`aligned`（垂直对齐）、`stroustrup`（左括号不换行）

27. **`fsharp_newline_before_multiline_computation_expression`**  
    多行计算表达式前换行（true/false，默认true）

---

### **[G-Research style]** 
28. **`fsharp_newline_between_type_definition_and_members`**  
    类型定义与成员间空行（true/false，默认true）

29. **`fsharp_align_function_signature_to_indentation`**  
    长函数签名对齐缩进（true/false，默认false）

30. **`fsharp_alternative_long_member_definitions`**  
    替代长成员定义格式（true/false，默认false）

31. **`fsharp_multi_line_lambda_closing_newline`**  
    多行lambda闭括号换行（true/false，默认false）

32. **`fsharp_experimental_keep_indent_in_branch`**  
    保持分支缩进（实验性，true/false，默认false）

33. **`fsharp_bar_before_discriminated_union_declaration`**  
    DU类型用例前强制加`|`（true/false，默认false）

注：标记为 **G-Research** 的配置项需整套使用以保持风格一致；

---

### **[Other]**
34. **`fsharp_blank_lines_around_nested_multiline_expressions`**  
    嵌套多行表达式周围空行（true/false，默认true）

35. **`fsharp_keep_max_number_of_blank_lines`**  
    保留最大连续空行数（≥1，默认100）

36. **`fsharp_experimental_elmish`**  
    Elmish风格列表格式化（实验性，true/false，默认false）

---

