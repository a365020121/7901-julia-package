# 7901-julia-package
This is a forecast Julia package

# How to use Julia to forecast timeseries data by Arima model

A famous and widely used statistical model is ARIMA. In R, there is a useful package “forecast” that you can use Arima model by a simple function “Arima()”. Unfortunately, we need to build the model by ourselves in Julia.
In this tutorial, we can know:
+	How to build ACF and PACF functions.
+ How to build Arima model in Julia.
+ About ARIMA model parameters and assumptions.
+	How to use this model with “timeseries” package to forecast data.

## ACF, PACF and differencing:
ARIMA model has three main parameters (p, d, q). 
Auto Regressive (AR(p)) model is referring to the use of past values in the regression equation. Parameter “p” is the number of lags used in the AR model. AR(2) model is the same as ARIMA(2, 0, 0) and can be represented as

![formula1](https://github.com/a365020121/7901-julia-package/blob/master/img/fomula1.png "AR model formula")

The parameter “d” represents the degree of differencing. The main target of differencing is to make the timeseries data stationary.
Moving Average (MA(q)) model is to use previous errors to predict future value. The parameter “q” is to determine the number of previous errors in the model. The model MA(2) can be represented as

![formula2](https://github.com/a365020121/7901-julia-package/blob/master/img/formula2.png "MA model formula")

First, we need to determine the parameters “p” and “q”. Therefore, we need ACF and PACF.
Autocorrelation Function (ACF) is the correlation of a signal with a delayed copy of itself as a function of delay. In other words, the autocorrelation function plot will let you know how the data is correlated with itself.
The Autocovariance Function at lag k is defined by

![formula3](https://github.com/a365020121/7901-julia-package/blob/master/img/formula3.png "ACF model formula")

The Autocorrelation Function at lag k is defined by

![formula4](https://github.com/a365020121/7901-julia-package/blob/master/img/formula4.png "ACF model formula")

In R, we can use “forecast” package to get ACF plot.
The data we used is the “AirPassengers”.

```R
  install.packages("forecast")
  install.packages("ggplot2")
  install.packages("tseries")
  library(forecast)
  library(ggplot2)
  library(tseries)
  
  data(AirPassengers) # load data
  AP<- AirPassengers
  plot(AP)
```

![pic1](https://github.com/a365020121/7901-julia-package/blob/master/img/pic1.png "AP plot")

```R
  AP
```

![pic2](https://github.com/a365020121/7901-julia-package/blob/master/img/pic2.png "AP")

We can use function “acf()” to get the ACF plot.

```R
  autoplot(acf(AP, lag.max=12, plot= FALSE))
```
![pic3](https://github.com/a365020121/7901-julia-package/blob/master/img/pic3.png "AP ACF")

We can set the parameter “plot = FALSE” to see the actual number of ACF.

```R
  acf(AP, lag.max = 12, plot = FALSE)
 
 >Autocorrelations of series ‘AP’, by lag

 0.0000 0.0833 0.1667 0.2500 0.3333 0.4167 0.5000 0.5833 0.6667 0.7500 0.8333 0.9167 1.0000 
 1.000  0.948  0.876  0.807  0.753  0.714  0.682  0.663  0.656  0.671  0.703  0.743  0.760 
```
The numbers below are the ACF of AirPassengers at lags 1 to 12.

```R
  1.000  0.948  0.876  0.807  0.753  0.714  0.682  0.663  0.656  0.671  0.703  0.743  0.760
```

Now we need to get the same results in Julia.

```Julia
using Pkg
using Plots
using CSV
Pkg.add("Plots")
Pkg.add("CSV")
Pkg.add("TimeSeries")

df = CSV.File("AirPassengers.csv") # load data

```
The data is shown as below
```Julia
144-element CSV.File{false}:
 [1949-01-01, 112]
 [1949-02-01, 118]
 [1949-03-01, 132]
 [1949-04-01, 129]
 [1949-05-01, 121]
 [1949-06-01, 135]
 [1949-07-01, 148]
 [1949-08-01, 148]
 [1949-09-01, 136]
 [1949-10-01, 119]
 [1949-11-01, 104]
 [1949-12-01, 118]
 [1950-01-01, 115]
 ⋮                
 [1960-01-01, 417]
 [1960-02-01, 391]
 [1960-03-01, 419]
 [1960-04-01, 461]
 [1960-05-01, 472]
 [1960-06-01, 535]
 [1960-07-01, 622]
 [1960-08-01, 606]
 [1960-09-01, 508]
 [1960-10-01, 461]
 [1960-11-01, 390]
 [1960-12-01, 432]
```

We also can use Timeseries package to help us read csv data and save the data as timearray:
```Julia
AP = readtimearray("AirPassengers.csv", format="yyyy-mm-dd";delim = ',')

144×1 TimeArray{Float64,2,Date,Array{Float64,2}} 1949-01-01 to 1960-12-01
│            │ #Passengers │
├────────────┼─────────────┤
│ 1949-01-01 │ 112.0       │
│ 1949-02-01 │ 118.0       │
│ 1949-03-01 │ 132.0       │
│ 1949-04-01 │ 129.0       │
│ 1949-05-01 │ 121.0       │
│ 1949-06-01 │ 135.0       │
│ 1949-07-01 │ 148.0       │
│ 1949-08-01 │ 148.0       │
│ 1949-09-01 │ 136.0       │
│ 1949-10-01 │ 119.0       │
│ 1949-11-01 │ 104.0       │
│ 1949-12-01 │ 118.0       │
   ⋮
│ 1960-02-01 │ 391.0       │
│ 1960-03-01 │ 419.0       │
│ 1960-04-01 │ 461.0       │
│ 1960-05-01 │ 472.0       │
│ 1960-06-01 │ 535.0       │
│ 1960-07-01 │ 622.0       │
│ 1960-08-01 │ 606.0       │
│ 1960-09-01 │ 508.0       │
│ 1960-10-01 │ 461.0       │
│ 1960-11-01 │ 390.0       │
│ 1960-12-01 │ 432.0       │
```

Then we need to get the values of the data.
```Julia
values(AP)

144×1 Array{Float64,2}:
 112.0
 118.0
 132.0
 129.0
 121.0
 135.0
 148.0
 148.0
 136.0
 119.0
 104.0
 118.0
 115.0
   ⋮  
 417.0
 391.0
 419.0
 461.0
 472.0
 535.0
 622.0
 606.0
 508.0
 461.0
 390.0
 432.0
 ```
 
 Now, we can create a function for ACF.
 
 ```Julia
 # acf plot function
# parameters: data, the data you want to caculate the acf
# parameters: k, how many lags do you want
# parameters: ifplot, true: plot the ACF values, false: return the ACF values
function plot_acf3(data,k, ifplot= false)
    
    data = values(data)
    acf = []
    n = length(data)
    avg = mean(data)
    
    for x in (1:k)
        
        rk = 0
        rk1 = 0
        rk2 = 0
        sum_rk1 = 0
        sum_rk2 = 0
        i = 1+x
        j = 1
        
        while i < n+1
            
            rk1 = (data[i]-avg) * (data[i-x]-avg)
            sum_rk1 = sum_rk1+rk1
            i = i+1
        end
        
        sum_rk1 = sum_rk1/n   
        while j < n+1
            
            rk2 = (data[j]-avg)^2
            sum_rk2 = rk2+sum_rk2
            j=j+1
            
        end  
        sum_rk2 = sum_rk2/n    
        rk = sum_rk1/sum_rk2
        rk = round(rk; digits =3 )  
        append!(acf, rk)
        
    end
    
    if ifplot == true
        
        return plot(acf, seriestype = :scatter)
        
    elseif ifplot == false
            
        return acf
    end    
end
```
Let us check if the function works.
```julia
# example
rk = plot_acf3(AP,12,true)
```
![pic4](https://github.com/a365020121/7901-julia-package/blob/master/img/pic4.png "Julia ACF")

```julia
# example
plot_acf3(AP,12,false)

12-element Array{Any,1}:
 0.948
 0.876
 0.807
 0.753
 0.714
 0.682
 0.663
 0.656
 0.671
 0.703
 0.743
 0.76 
```
The function works!

---
Questions:
+ question1
I want my function to do that
```Julia
plot_acf3(AP,12, ifplot = false)
```
but it will get a error:
```Julia
function plot_acf3 does not accept keyword arguments
```
The error appear beacuse i add ifplot in my arguments.
How to solve the problem?

