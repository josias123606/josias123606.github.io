---
title: "Series de Fourier y Convergencia"
date: 2026-05-21
categories: [Fourier, Análisis]
---

Las **Series de Fourier** son una de las herramientas más poderosas del análisis matemático. Permiten representar funciones periódicas como sumas infinitas de senos y cosenos. En esta entrada exploramos su definición, propiedades y algunos resultados de convergencia fundamentales.

## Definición

Sea $f: \mathbb{R} \to \mathbb{R}$ una función periódica con periodo $2\pi$ e integrable en $[-\pi, \pi]$. Su **serie de Fourier** se define como:

$$f(x) \sim \frac{a_0}{2} + \sum_{n=1}^{\infty} \left( a_n \cos(nx) + b_n \sin(nx) \right)$$

donde los **coeficientes de Fourier** son:

$$a_n = \frac{1}{\pi} \int_{-\pi}^{\pi} f(x)\cos(nx)\, dx, \quad n = 0, 1, 2, \ldots$$

$$b_n = \frac{1}{\pi} \int_{-\pi}^{\pi} f(x)\sin(nx)\, dx, \quad n = 1, 2, 3, \ldots$$

## Forma Compleja

Usando la fórmula de Euler $e^{inx} = \cos(nx) + i\sin(nx)$, la serie se puede escribir de forma compacta como:

$$f(x) \sim \sum_{n=-\infty}^{\infty} c_n \, e^{inx}$$

donde el coeficiente complejo es:

$$c_n = \frac{1}{2\pi} \int_{-\pi}^{\pi} f(x)\, e^{-inx}\, dx$$

La relación entre los coeficientes real y complejo es: $c_n = \frac{a_n - ib_n}{2}$ para $n \geq 1$.

## Ejemplo: La Función Diente de Sierra

Consideremos $f(x) = x$ en $(-\pi, \pi)$, extendida periódicamente. Sus coeficientes son:

$$a_n = 0 \quad \text{(la función es impar)}$$

$$b_n = \frac{2(-1)^{n+1}}{n}$$

Por lo tanto:

$$x \sim 2\sum_{n=1}^{\infty} \frac{(-1)^{n+1}}{n} \sin(nx) = 2\left(\sin x - \frac{\sin 2x}{2} + \frac{\sin 3x}{3} - \cdots\right)$$

## Convergencia: Teorema de Dirichlet

> **Teorema (Dirichlet, 1829):** Si $f$ es periódica, acotada y tiene un número finito de máximos, mínimos y discontinuidades en $[-\pi, \pi]$, entonces su serie de Fourier converge a:
> $$\frac{f(x^+) + f(x^-)}{2}$$
> en cada punto $x$. En los puntos de continuidad, converge a $f(x)$.

## Identidad de Parseval

Una consecuencia importante es la **Identidad de Parseval**, que relaciona la energía de la función con sus coeficientes:

$$\frac{1}{\pi}\int_{-\pi}^{\pi} |f(x)|^2\, dx = \frac{a_0^2}{2} + \sum_{n=1}^{\infty}\left(a_n^2 + b_n^2\right)$$

En notación compleja:

$$\frac{1}{2\pi}\int_{-\pi}^{\pi} |f(x)|^2\, dx = \sum_{n=-\infty}^{\infty} |c_n|^2$$

## Una Suma Notable

Aplicando Parseval a la función diente de sierra $f(x)=x$ obtenemos:

$$\frac{1}{\pi}\int_{-\pi}^{\pi} x^2\, dx = \sum_{n=1}^{\infty} \frac{4}{n^2}$$

Calculando la integral del lado izquierdo:

$$\frac{2\pi^2}{3} = 4\sum_{n=1}^{\infty} \frac{1}{n^2}$$

Lo que nos da el famoso resultado de Euler:

$$\sum_{n=1}^{\infty} \frac{1}{n^2} = \frac{\pi^2}{6}$$

Este es uno de los resultados más elegantes del análisis matemático, conectando las series de Fourier con la función zeta de Riemann $\zeta(2)$.

## Fenómeno de Gibbs

Cerca de una discontinuidad, la serie de Fourier no converge uniformemente. El error máximo tiende a aproximadamente el $\mathbf{9\%}$ del salto de la discontinuidad, sin importar cuántos términos se sumen. Este comportamiento se llama **Fenómeno de Gibbs** y tiene implicaciones importantes en procesamiento de señales.

Matemáticamente, el sobreimpulso máximo converge a:

$$\frac{1}{\pi}\int_0^{\pi} \frac{\sin t}{t}\, dt - \frac{1}{2} \approx 0.0895$$

es decir, alrededor del $8.95\%$ del salto total.
