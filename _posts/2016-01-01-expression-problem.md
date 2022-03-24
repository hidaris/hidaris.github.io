---
layout: single
title:  "Expression Problem 的另一个解法"
date:   2016-01-01 15:33:01 +0800
categories: PL
---
>The Expression Problem is a new name for an old problem. The goal is to define a datatype by cases, where one can add new cases to the datatype and new functions over the datatype, without recompiling existing code, and while retaining static type safety (e.g., no casts). — Philip Wadler

首先，为了更直观的理解这个问题，我们来看一段用 Typed Racket 写的例子。

```racket
(define-type Expr (U Lit Add))
(struct Lit ([i : Integer]) #:transparent)
(struct Add ([e1 : Expr] [e2 : Expr]) #:transparent)

(: e : Expr)
(define e (Add (Lit 1) (Add (Lit 1) (Lit 1))))

(: eval : Expr -> Integer)
(define (eval e)
  (match e
    [(Lit i) i]
    [(Add e1 e2)
     (+ (eval e1) (eval e2))]))
```

在这段代码中，我们定义了一个表达式类型 Expr，以及一个基于 Expr 类型进行模式匹配的函数 eval。然后，我们另写一个基于 Expr 类型的函数 view，它的作用是查看对应类型的字符串表示，定义如下：

```racket
(: view : Expr -> String)
(define (view e)
  (match e
    [(Lit i) (number->string i)]
    [(Add e1 e2) (string-append (view e1) "+" (view e2))]))
```

通过这两个例子，我们可以看出添加一个函数并不会影响之前的函数，但是添加一个 construct 的情况就不一样了

```racket
(define-type Expr (U Lit Add Mul))
(struct Lit ([i : Integer]) #:transparent)
(struct Add ([e1 : Expr] [e2 : Expr]) #:transparent)
(struct Mul ([e1 : Expr] [e2 : Expr]) #:transparent)
```

因为 Expr 中多了一个 Mul 类型，所以以 Expr 为参数类型的函数全都会受到影响。Why？因为模式匹配并没有 handle 那个新类型，所以为了修复这个问题，我们得改动所有的这些函数来添加对 Mul 类型的处理。

```racket
(: eval : Expr -> Integer)
(define (eval e)
  (match e
    [(Lit i) i]
    [(Add e1 e2)
     (+ (eval e1) (eval e2))]
    [(Mul e1 e2)
     (* (eval e1) (eval e2))]))

(: view : Expr -> String)
(define (view e)
  (match e
    [(Lit i) (number->string i)]
    [(Add e1 e2) (string-append (view e1) "+" (view e2))]
    [(Mul e1 e2) (string-append (view e1) "*" (view e2))]))
```

这个问题就是经典的表达式问题，按照传统的抽象方法，我们会在 FP 中添加类型或者 OOP 中添加函数时遇到这个问题，我们先讨论 FP 的情况如何解决，然后进一步拓展到 OOP 中如何解决，对于这个问题，有许多先辈提出了一些解法，比如 Tagless-final，解决思路很有趣，但是今天我们来重新思考是否有其他解法，这个思考的过程比直接学会一个解法更重要。

我们定义的 Expr 类型是一个 Union type，在范畴论中，我们称之为 Coprodut， 从 wiki 中我们可以看出这种 关系和 Persistent Linked_list 是一种近似同构的关系。这里我开个玩笑，从中我们得到一个解决表达式问题的思路：Persistent Union Type，当然这是我生造的一个名词:) 在继续往下读前，请先阅读一遍我给出的两篇 wiki，内容不是很难理解，大概需要10分钟左右的时间。

OK，我们知道了 union type 和 persistent linked_lists 是一种同构关系，在链表前增加节点，对应到类型中，就是在原有的类型上并上一个新的 constructor，由于我们未丢失原来的结构，所以新链表与原有链表共享所有的旧节点，我们新的类型与原来的类型也共享同样的 constructor，对新的链表或类型进行的变换（函数）等价于对旧结构进行变换并加上新结构的变换，这里看两个相似的例子感受一下： 简单算术运算

```racket
(x+y+z)*2 = x*2+y*2+z*2
          = (x+y)*2+z*2
```

链表操作

```racket
(map (curry + 2) (quote 1 2 3)) = (cons (+ 2 1)
                                    (cons (+ 2 2)
                                      (cons (+ 2 3)
                                            empty)))
                                = (cons (+ 2 1) (map (curry + 2) (quote 2 3)))
```

那么，我们来看看以这个思路，如何给 Expr 类型添加 Mul 拓展

```racket
(define-type Expr (U Lit Add))
(struct Lit ([i : Integer]) #:transparent)
(struct Add ([e1 : Expr] [e2 : Expr]) #:transparent)
(define-type Expr+Mul (U Expr Mul))
(struct Mul ([e1 : Expr] [e2 : Expr]) #:transparent)
```

可以将类型进一步展开

```racket
;; Expr
(U Lit Add)
;; Expr+Mul
(U Lit Add Mul)
```

类型是抽象值的集合，集合的并可以看做“加法”，也就是说我们直接在类型上做加法，然后可以看看相应的函数拓展：

```racket
(define-type Expr (U Lit Add))
(struct Lit ([v : Integer]) #:transparent)
(struct Add ([e1 : Expr] [e2 : Expr]) #:transparent)
(define-predicate Expr? Expr)

(: eval-add : Exp -> Integer)
(define (eval-add e)
  (match e
    [(Lit n) n]
    [(Add e1 e2) (+ (eval-add e1)
                    (eval-add e2))]))

(define-type Expr+Mul (U Exp Mul))
(struct Mul ([e1 : Expr+Mul] [e2 : Expr+Mul]) #:transparent)

(: eval-mul : Exp+Mul -> Integer)
(define (eval-mul e)
  (cond
    [(Exp? e) (eval-add e)]
    [else
     (match e
       [(Mul e1 e2) (- (eval-mul e1)
                       (eval-mul e2))])]))
```

所以，问题到这里就解决了吗？我们将添加 construct 的操作看作类型上的加法，相应的函数也可以复用旧有的函数，一切似乎都 OK，但是这么做还有一个问题，如果我们给 Add 传递一个 Mul 类型的参数，会发现在函数eval-add 中没有 handle 这个类型。因为 Add 接受的是 Expr 类型，所以只接受 Val 和 Add，而扩展后的 Expr 名变成了 Expr+Mul，我们希望的是能够扩展 Expr 类型而不改变它的名字，这听起来像什么？没错，像在面向对象中声明一个接口，然后给它添加一个新的实现，这在 FP 中叫做 subtype。因此，我们可以将 Expr 的定义以这种方式修改下：

```racket
(struct Expr ())
(struct Val Expr ([v : Integer]) #:transparent)
(struct Add Expr ([e1 : Expr] [e2 : Expr]) #:transparent)
```

上述的新做法是定义一个 Expr 类型，然后将 Val，Add 分别作为它的 subtype，现在可以为它定义 eval 函数了

```racket
(: eval : Exp -> Integer)
(define (eval e)
  (match e
    [(Val n) n]
    [(Add e1 e2) (+ (eval e1)
                    (eval e2))]))
;; test
(= (eval (Add (Val 2) (Val 3))) 5) ; return true
```

定义了 eval 后，我们得扩展 Expr，给它添加一个新的 subtype，Mul：

```racket
(struct Mul Expr ([e1 : Expr] [e2 : Expr]) #:transparent)
```

最终，我们要扩展 eval，使它能够 handle Mul 类型，如何做呢？如果采取我们之前的做法

```racket
(: evalM (-> Expr Integer))
(define (evalM e)
  (cond
    [(Expr? e) (eval e)]
    [(Mul? e) (- (eval (Mul-e1 e))
                 (eval (Mul-e2 e)))]))
```

TR 会给你一个error: match: no matching clause for (Mul ...，因为现在 Expr 包括了 Mul，所以 Mul 这一行永远执行不到，那么我们调整一下cond分支的顺序，先处理 Mul 然后再处理剩下的 Val 或者 Add：

```racket
(: evalM (-> Expr Integer))
(define (evalM e)
  (cond
    [(Mul? e) (- (eval (Mul-e1 e))
                 (eval (Mul-e2 e)))]
    [(Expr? e) (eval e)]))
```

现在，我们可以对(Mul (Add (Val 3) (Val 2)) (Val 1))这样的类型求值了，但是如果 Mul 出现在 Add 的参数中，则求值依旧会失败，因为匹配到 Add 时，我们使用的是 eval 方法，而 eval 不能 handle Mul 类型，所以我们需要一个可扩展的函数,Racket 自身并没有提供这种函数的定义方式。我们可以安装extensible-functions这个包，本来我也不知道这个包的存在，在写了这篇博客的早期，我大体想过如何解决这个问题，就是利用 Racket 的Macro在compile time将原来的函数 eval 复制到新函数 evalM 中，并且统一名字为 evalM，也就是说虽然我们在用户代码看到的是简单的对 eval 的扩展，但实际上这个函数已经被转换成：

```racket
(: eval : Expr -> Integer)
(define (eval e)
  (match e
    [(Val n) n]
    [(Add e1 e2) (+ (eval e1)
                    (eval e2))]
    [(Mul e1 e2) (- (eval e1)
                    (eval e2))]))
```

因为代码的转换以及类型的检查都在compile time发生，所以我们并没有破坏类型的安全性。但是由于我不太喜欢调试宏，就一直坑在这儿，直到前不久看到 @leafac 的这个包，才发现原来 Maryland University 也有人有过相似的想法，现在来看看最终的解法：

```racket
(define/match/extensible (eval e)
  : (-> Expr Integer)
  [((Val n)) n]
  [((Add e1 e2)) (+ (eval e1)
                    (eval e2))])
(define/match/extension/eval
  [((Mul e1 e2)) (- (eval e1)
                    (eval e2))])
;; test
(= (eval (Add (Mul (Val 3) (Val 2)) (Val 1))) 2)
```

本篇到这里就结束了，如果你想进一步了解 EP 的其他解法，可以顺着 wiki 看看已经发表的相关论文，如果对于如何在 OOP 中解决这个问题感兴趣请阅读我的另一篇文章 From FP to OOP, a.k.a Patterns。
