Intro to Neural Nets
======================


o
 \ w1
   o
o / w2

z = f(w.x + b)

f ==> "activiation function"

each node's output is its activation function applied to its input

## Activation functions

Rectifier: f(t) = max { 0, t }

Sigmoid: f(t) = 1/(1+e^-t)

Tanh: f(t) = 2 * sig(2t) - 1


inp1 --> z1 --> z3 \
     /\     /\      output
inp2 --> z2 --> z4 /

z3 = f3((z1) * w  + b3)
       ((z2)          )

Regression:

Y = W * Z

Classification:

Y =   exp(W * Z)
    --------------
    sum(exp(W * Z)

Read about Backpropogation
