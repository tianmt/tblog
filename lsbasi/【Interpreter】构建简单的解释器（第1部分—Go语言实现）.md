@[toc]
# 【Interpreter】构建简单的解释器（第1部分—Go语言实现）



## 一、描述



> 1. 能够计算两个个位整数的和，如 3 + 5。



## 二、代码



```go
// 使用 Go 简单实现

package main

import (
	"bufio"
	"fmt"
	"os"
	"strings"
	"unicode"
)

type TOKENTYPE int

const (
	TYPE_NONE TOKENTYPE = iota
	TYPE_INTEGER
	TYPE_PLUS
	TYPE_EOF
)

type Token struct {
	token_char interface{}
	token_type TOKENTYPE
}

type Interpreter struct {
	text          []rune
	pos           int
	current_token Token
}

func RuneToInt(c rune) (int, bool) {
	if unicode.IsDigit(c) {
		return int(c - '0'), true
	}
	return 0, false
}

func (interpreter *Interpreter) Eat(token_type TOKENTYPE) {
	if interpreter.current_token.token_type == token_type {
		interpreter.current_token = interpreter.GetNextToken()
		return
	}

	panic("type error.")
}

func (interpreter *Interpreter) GetNextToken() Token {
	text := interpreter.text

	if interpreter.pos <= len(text)-1 {

		current_char := interpreter.text[interpreter.pos]

		if v, ok := RuneToInt(current_char); ok {
			token := Token{v, TYPE_INTEGER}
			interpreter.pos++
			return token
		}

		if '+' == current_char {
			token := Token{current_char, TYPE_PLUS}
			interpreter.pos++
			return token
		}
	}

	return Token{TYPE_EOF, 0}
}

func (interpreter *Interpreter) Expr() {

	defer func() {
		if r := recover(); r != nil {
			fmt.Println("[ERROR] ", r)
		}
	}()

	interpreter.current_token = interpreter.GetNextToken()

	left := interpreter.current_token
	interpreter.Eat(TYPE_INTEGER)

	// operator := interpreter.current_token
	interpreter.Eat(TYPE_PLUS)

	right := interpreter.current_token

	result := left.token_char.(int) + right.token_char.(int)

	fmt.Println("> ", result)
}

func main() {
	fmt.Println("------------------ <PART-1> ------------------")
	reader := bufio.NewReader(os.Stdin)

	for {
		fmt.Print("[CALC]-> ")

		text, _ := reader.ReadString('\n')
		my_text := strings.TrimSpace(strings.TrimSuffix(strings.ToLower(text), "\n"))

		if len(my_text) == 0 {
			continue
		}

		if my_text == "exit" {
			fmt.Println("-------------------程序已退出-------------------")
			fmt.Println("------------------- <END> --------------------")
			os.Exit(0)
		}

		// 处理字符串
		interpreter := Interpreter{[]rune(my_text), 0, Token{}}
		interpreter.Expr()
	}
}

```



## 三、运行结果



```shell
------------------ <PART-1> ------------------
[CALC]-> 1+2
>  3
[CALC]-> 1+9
>  10
[CALC]-> 3+5
>  8
[CALC]-> 11+1
[ERROR]  type error.
[CALC]-> 1 +2 
[ERROR]  type error.
[CALC]-> exit
-------------------程序已退出-------------------
------------------- <END> --------------------
```



------

**——2019-01-15——**