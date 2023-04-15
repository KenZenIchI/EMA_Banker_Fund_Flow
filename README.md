// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © CryptoButgi_Bot





//@version=4
strategy("Backtest single EMA cross", overlay=true)



//functions
xrf(values, length) =>
    r_val = float(na)
    if length >= 1
        for i = 0 to length by 1
            if na(r_val) or not na(values[i])
                r_val  :=  values[i]
                r_val
    r_val

xsa(src,len,wei) =>
    sumf = 0.0
    ma = 0.0
    out = 0.0
    sumf  :=  nz(sumf[1]) - nz(src[len]) + src
    ma  :=  na(src[len]) ? na : sumf/len
    out  :=  na(out[1]) ? ma : (src*wei+out[1]*(len-wei))/len
    out
    
//set up a simple model of banker fund flow trend	
fundtrend = ((3*xsa((close- lowest(low,27))/(highest(high,27)-lowest(low,27))*100,5,1)-2*xsa(xsa((close-lowest(low,27))/(highest(high,27)-lowest(low,27))*100,5,1),3,1)-50)*1.032+50)
//define typical price for banker fund
typ = (2*close+high+low+open)/5
//lowest low with mid term fib # 34
lol = lowest(low,34)
//highest high with mid term fib # 34
hoh = highest(high,34)
//define banker fund flow bull bear line
bullbearline = ema((typ-lol)/(hoh-lol)*100,13)
//define banker entry signal
bankerentry = crossover(fundtrend,bullbearline) and bullbearline<25

//banker fund entry with yellow candle
// plotcandle(0,50,0,50,color=bankerentry ? color.new(color.yellow,0):na)

//banker increase position with green candle
// plotcandle(fundtrend,bullbearline,fundtrend,bullbearline,color=fundtrend>bullbearline ? color.new(color.green,0):na)

//banker decrease position with white candle
// plotcandle(fundtrend,bullbearline,fundtrend,bullbearline,color=fundtrend<(xrf(fundtrend*0.95,1)) ? color.new(color.white,0):na)

//banker fund exit/quit with red candle
// plotcandle(fundtrend,bullbearline,fundtrend,bullbearline,color=fundtrend<bullbearline ? color.new(color.red,0):na)

//banker fund Weak rebound with blue candle
// plotcandle(fundtrend,bullbearline,fundtrend,bullbearline,color=fundtrend<bullbearline and fundtrend>(xrf(fundtrend*0.95,1)) ? color.new(color.blue,0):na)

//overbought and oversold threshold lines
// h1 = hline(80,color=color.red, linestyle=hline.style_dotted)
// h2 = hline(20, color=color.yellow, linestyle=hline.style_dotted)
// h3 = hline(10,color=color.lime, linestyle=hline.style_dotted)
// h4 = hline(90, color=color.fuchsia, linestyle=hline.style_dotted)
// fill(h2,h3,color=color.yellow,transp=70)
// fill(h1,h4,color=color.fuchsia,transp=70)

// alertcondition(bankerentry, title='Alert on Yellow Candle', message='Yellow Candle!')
// alertcondition(fundtrend>bullbearline, title='Alert on Green Candle', message='Green Candle!')
// alertcondition(fundtrend<(xrf(fundtrend*0.95,1)), title='Alert on White Candle', message='White Candle!')
// alertcondition(fundtrend<bullbearline, title='Alert on Red Candle', message='Red Candle!')
// alertcondition(fundtrend<bullbearline and fundtrend>(xrf(fundtrend*0.95,1)), title='Alert on Blue Candle', message='Blue Candle!')





qty = input(100000, "Buy quantity")

testStartYear = input(2019, "Backtest Start Year")
testStartMonth = input(1, "Backtest Start Month")
testStartDay = input(1, "Backtest Start Day")
testStartHour = input(0, "Backtest Start Hour")
testStartMin = input(0, "Backtest Start Minute")
testPeriodStart = timestamp(testStartYear, testStartMonth, testStartDay, testStartHour, testStartMin)
testStopYear = input(2099, "Backtest Stop Year")
testStopMonth = input(1, "Backtest Stop Month")
testStopDay = input(30, "Backtest Stop Day")
testPeriodStop = timestamp(testStopYear, testStopMonth, testStopDay, 0, 0)
testPeriodBackground = input(title="Color Background?", type=input.bool, defval=true)
testPeriodBackgroundColor = testPeriodBackground and time >= testPeriodStart and time <= testPeriodStop ? 
   #00FF00 : na
testPeriod() =>
    time >= testPeriodStart and time <= testPeriodStop ? true : false


ema1 = input(10, title="Select EMA 1")
ema2 = input(20, title="Select EMA 2")
ema3 = input(200, title="Select EMA 3")
ema3Line = ema(close, ema3)
expo = ema(close, ema1)
ma = ema(close, ema2)

avg_1 = avg(expo, ma)
s2 = cross(expo, ma) ? avg_1 : na
plot(ema3Line, style=plot.style_line, linewidth=3, color=color.red, transp=0)

p1 = plot(expo, color=#00FFFF, linewidth=2, transp=0)
p2 = plot(ma, color=color.orange, linewidth=2, transp=0)
fill(p1, p2, color=color.white, transp=80)

longCondition = crossover(expo, ma)

shortCondition = crossunder(expo, ma)

stopLong = lowest(5)
stopShort = highest(5)
var SStopLong = 0.0
var SStopShort = 0.0

// if testPeriod()
//     strategy.entry("Long", strategy.long, when=longCondition)
//     strategy.entry("Short", strategy.short, when=shortCondition)
var LongEntry = 0.0
var ShortEntry = 0.0
if longCondition and fundtrend>bullbearline and fundtrend[0] > 79.0 and close > ema3Line[1] and LongEntry == 0.0
    strategy.entry('Long', strategy.long)
    LongEntry := 1.0
    SStopLong := stopLong[0]

if LongEntry == 1.0 and close >= (1.005 * strategy.position_avg_price) and strategy.position_size > 0.0
    strategy.close('Long')
    LongEntry := 0.0

if close < SStopLong and LongEntry == 1.0 and strategy.position_size > 0.0
    strategy.close('Long')
    LongEntry := 0.0
    SStopLong := 0.0


if shortCondition and fundtrend<bullbearline and fundtrend[0] < 21.0 and close < ema3Line[1] and ShortEntry == 0.0
    strategy.entry('Short', strategy.short)
    ShortEntry = 1.0
    SStopShort := stopShort[0]
if close <= (0.995 * strategy.position_avg_price) and strategy.position_size < 0.0
    strategy.close('Short')
    ShortEntry := 0.0
if close > SStopShort and strategy.position_size < 0.0
    strategy.close('Short')
    ShortEntry := 0.0
    SStopShort := 0.0
// plotshape(longCondition, title = "Buy Signal", text ="BUY", textcolor =#FFFFFF , style=shape.labelup, size = size.normal, location=location.belowbar, color = #1B8112, transp = 0)
// plotshape(shortCondition, title = "Sell Signal", text ="SELL", textcolor = #FFFFFF, style=shape.labeldown, size = size.normal, location=location.abovebar, color = #FF5733, transp = 0)











//ELEMENTS ==>{
    // 
//For_Buy
// GreanCandle =
// EmaCrossUpper = 
// TrendOSilatorGreenCandle = 
// TrendOsilatorUpperThanEighty = 

