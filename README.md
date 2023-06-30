# Mitsuba with debiasing methods

This project is aimed for implementing debiasing methods for volumetric rendering, based on the `cmake-version` fork of `mitsuba-v0.6`.

## 1. building

**Do not modify the directory `build` !!**  , where many specific config files locate. Instead, create another directory like `cbuild` to place building results. 

To build the project please look into 

[**`BUILD.md`**]: ./BUILD.md

 for further guide.

Mitsuba0.6 is integrated with **GUI**, so after you successfully build the project you can use the **GUI** to test with some specific scenes. 

## 2. testing

I have modified `./src/medium/heterogeneous.cpp` ,adding into debiasing methods for the evaluation of transmittance. 

Two methods appended to the class `heterogeneous` are respectively

```C++
float Heterogeneous::DebaisedTransmittance(const Ray &ray, Sampler* sampler) const;
```

and

```C++
bool DebiasedInvertDensityIntegral(const float psi, const Ray &ray, float desiredDensity, float &integratedDensity, float &t, float &densityAtMinT,
            float &densityAtT) const;
```



The first one calculates the transmittance between `ray.mint` and `ray.maxt` in an unbiased way.
$$
\begin{equation}
\begin{split}
tr & = g[\int^{t_{max}}_{t_{min}}\mu(t)dt] \\
\end{split}
\end{equation}
$$
The latter determines the scattering location in media during volumetric path-tracing,  i.e. solves the following equation in an unbiased way.
$$
\begin{equation}
g[-\int^{t}_{t_{min}}\mu(t')dt'] = tr_0
\end{equation}
$$


Mostly, function $g$ is selected as $g(x) = e^x$, while the project also includes function introduced by Davis and Mineev-Weinstein :
$$
\begin{equation}
g(x) = (1+F^\beta C^{1+\beta})^{-\frac{F^{1-\beta}}{ C^{1+\beta}}}\quad \beta,C\in[0,1]
\end{equation}
$$

There are two kinds of debiasing methods included in the project, `Taylor series method` and `Telescope method` respectively. To choose between them or choose not to adopt unbiased method (which is by default), one can add one of the terms below within the scope of `<medium> <medium/>`
```xml
<string name="debiasedMethod" value = "nodebiase" />
<string name="debiasedMethod" value = "telescope" />
<string name="debiasedMethod" value = "taylor" />
```

Further more, to choose between function $(3)$ and $e^x$, one can add one of 
the terms below within the scope of `<medium> <medium/>`
```xml
<string name="transmitteFunctionType" value = "exponential" />
<string name="transmitteFunctionType" value = "davis-mineev" />
```
and terms
```xml
<float name="C" value = "0.7" />
<float name="beta" value = "0.4" />
```
to set the value of `C` and `beta`.