# tesla-quant
 from quantopian.algorithm import attach_pipeline, pipeline_output
 from quantopian.pipeline import Pipeline
 #from quantopian.research import run_pipeline
 from quantopian.pipeline.data.user_5e3da1b96dd8c20046f64630 import tsla_sent as tsent
 from quantopian.pipeline.factors import AverageDollarVolume, SimpleMovingAverage
 
 from quantopian.pipeline.factors import CustomFactor
 import numpy as np

##----------------------------------------------------------------------------##

# Initialize & schedule function

def initialize(context):
    # Import Stocks
    context.tsla = sid(39840)
    context.spy = sid(8554)

    
    # Scheduling the Moving Average Crossover Function
    schedule_function(rebalance,date_rules.every_day(),time_rules.market_open())
    # Scheduling the Sentiment Function
    schedule_function(sentiment,date_rules.every_day(),time_rules.market_open())
    # Recording the Share Prices
    schedule_function(record_vars,date_rules.every_day(),time_rules.market_open())


# Moving Average Crossover Function

def rebalance(context, data):
    t = context.tsla
    #s = context.spy
    #cur_price = data.current(t,'price')
    #current_positions = context.portfolio.positions[t].amount
    #cash = context.portfolio.cash
    
    # Short-term Moving Average (50 day)
    price1 = data.history(t,'price',50,'1d')
    # Longer-term Moving Average (100 day)
    price2 = data.history(t,'price',200,'1d')

    
    # Short-term moving-average and std-ST
    avg1 = price1.mean()
    #std1 = price1.std()
    
    # Long-term moving-average and std-LT
    avg2 = price2.mean()
    #std2 = price2.std()
 
    
    ##-Applying the Moving Average crossover test-##
    if avg1 > avg2:
        #order_target_percent(t,1.0)
        order(t,+1000)
        log.info('Buying shares')
    elif avg1 < avg2:
        order(t,-500)
        log.info('Selling shares')
    else:
        pass
    
    
    
# Sentiment Function

def sentiment(context, data):
    t = context.tsla

    # Define variables
    context.longs = ['sent_score' in data['tsent'] < 'average_1' in data['tsent'] and 
                     'percent_change' in data['tsent'] < 0.10]
    context.shorts = ['sent_score' in data['tsent'] > 'average_1' in data['tsent'] and 
                      'percent_change' in data['tsent'] > 0.10]
 

    ##-Applying the Sentiment test-##
    if context.longs:
        order(t,+1000)
        log.info('Buying shares')
    elif context.shorts:
        order(t,-500)
        log.info('Selling shares')
    else:
        pass

    
# Record Share Prices of Tesla and SPY    

def record_vars(context,data):
    record(tsla_close=data.current(context.tsla,'close'))
    record(spy_close=data.current(context.spy,'close'))
