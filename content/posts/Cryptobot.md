---
title: "Cryptobot: A Cryptocurrency Trading Algorithm For The Binance Exchange Platform"
date: 2020-08-08T14:45:25Z
---


This is an algorithmic trading bot I’ve been using on the Binance cryptocurrency exchange. So far I have only used it with Ethereum/USDT.

Link to full repo after the code.

&nbsp;
&nbsp;

```Python

from binance.client import Client
import requests
import smtplib
from email.message import EmailMessage
import loguru
import time
import random
import os




class CryptoBot(Client):
    
    
    

    def __init__(self, currencies = None, email = None, **kwargs):
        super().__init__(**kwargs)

        self.currencies = currencies
        self.email = email
        
        self.pair = self.currencies["trading"] + self.currencies["wallet"]

        self.buy_in = {}
        
    
    
    
    @property 
    def funds(self):

        complete_funds = round(float(self.get_asset_balance(
            asset=self.currencies["wallet"])["free"]), 2)

        if complete_funds:
            return complete_funds


    
    
    @property
    def liquidity_check(self):

        if not self.funds > 1.0:

            self.email["subject"] = (
                "TRADING BALANCE TOO LOW")
            
            self.email["content"] = (
                """Trading bot reports a balance of zero. """ 
                """The bot will remain inactive until it """
                """can access adequate funds.""")

            self.notifier()

            while True:
                
                if not self.funds > 1.0:
                    time.sleep(600)
                else:  
                    break

            self.email["subject"] = (
                "TRADING BALANCE INCREASED")
            
            self.email["content"] = (
                """Trading bot reports it now has access to funds. """ 
                """The bot will now resume trade.""")

            self.notifier()

            return False

        return True
 
    
    

    @property   
    def order_check(self):


        if self.get_open_orders():

            self.email["subject"] = (
                "WAITING ON OPEN ORDERS")
            
            self.email["content"] = (
                """Trading bot has detected open orders. """ 
                """It will wait for all open orders to settle """
                """before it resumes operating.""")

            self.notifier()

            while True:

                if self.get_open_orders():
                    time.sleep(600)
                else:
                    break

            self.email["subject"] = (
                "OPEN ORDERS SETTLED. TRADING RESUMED")
            
            self.email["content"] = (
                """Trading bot reports that all open orders """ 
                """have been settled. It will now resume trade.""")

            self.notifier()

            return False

        return True


    @property
    def trend(self):

        candles = requests.get(
        f"https://api.binance.com/api/v3/klines?symbol={self.pair}&interval=1m&limit=60").json()

        old_price = float(candles[0][1])
        new_price = float(candles[-1][4])

        difference = (new_price - old_price) / old_price * 100

        return difference




    @property
    def circuit_breaker(self):

        if self.trend < -3.0:

            self.email["subject"] = (
                "CIRCUIT BREAKER IMPOSED")
            
            self.email["content"] = (
                """Circuit breaker imposed due to """ 
                """market losses.""")

            self.notifier()

            while True:

                if not price_movement > 2.0:
                    time.sleep(600)
                else:
                    break
                
            self.email["subject"] = (
                        "CIRCUIT BREAKER LIFTED")
                    
            self.email["content"] = (
                """Circuit breaker lifted due to """ 
                """price recovery.""")

            self.notifier()

            return False

        return True

    

    def buy(self):
        
        rate = round(float(self.get_symbol_ticker(symbol = self.pair)["price"]), 2)
        units = round(self.funds / rate - 0.0001, 5)

        self.buy_in = self.order_limit_buy(
            symbol = self.pair,
            price = str(rate),
            quantity = str(units))

        
        while True:

            if not self.get_order(
                orderId = self.buy_in["orderId"], 
                symbol = self.pair)["status"] == "FILLED":

                time.sleep(30)
            
            else:
                break



    def sell(self):
        
        time.sleep(5)
        
        cost = round(float(self.buy_in["price"]) * 1.015, 2)

        cashout = self.order_limit_sell(
            symbol = self.buy_in["symbol"],
            quantity = round(float(self.buy_in["origQty"]) - 0.0001, 5),
            price = str(cost))

        while True:

            if not self.get_order(
                orderId = cashout["orderId"], 
                symbol = cashout["symbol"]) == "FILLED":

                time.sleep(30)
            else:
                break

        self.buy_in = {}

 
    

    def trade(self):

        while True:
            if all([
                self.order_check,
                self.liquidity_check, 
                self.circuit_breaker]):
                
                self.buy()
                self.sell()
             



    def notifier(self):

        notification = EmailMessage()
        
        notification["To"] = self.email["destination"]
        notification["From"] = self.email["origin"]
        notification["Subject"] = self.email["subject"]
        notification.set_content(self.email["content"])
        
        with smtplib.SMTP_SSL("smtp.gmail.com") as mailboy:
            mailboy.login(self.email["username"], self.email["password"])
            mailboy.send_message(notification)




    def event_logger(self, event):
        
        logger = loguru.logger

        logger.add(
            sink = "events.log", 
            level = "WARNING", 
            format = """\n\n\n\n{level} {time: {time:DD-MM-YYYY HH:mm:ss}}\n"""
                     """Elapsed Time: {elapsed}\n"""
                     """File: {file}\n"""
                     """Message: {message}""")
        
        logger.exception(event)
        
        self.email["subject"] = "EXCEPTION ENCOUNTERED"
        self.email["content"] = str(event)

        self.notifier()




if __name__ == "__main__":
    

    key = os.environ.get("BINANCE_KEY")

    secret = os.environ.get("BINANCE_SECRET")
    
    ethereum = CryptoBot(
        api_key = key, 
        api_secret = secret,
        currencies = {
            "wallet": "USDT",
            "trading": "ETH"},
        email = {
            "origin": os.environ.get("EMAIL_ORIG"),
            "destination": os.environ.get("EMAIL_DEST"),
            "subject": None,
            "content": None,
            "username": os.environ.get("EMAIL_USER"),
            "password": os.environ.get("EMAIL_PASS")})
        
   
    while True:
        
        try:
            
            ethereum.trade()
        
        except Exception as event:
            
            ethereum.event_logger(event)
            time.sleep(600)
        
  

```




&nbsp;
&nbsp;


[Full code repo available on Github](https://github.com/Capybasilisk/Cryptobot)

&nbsp;

Related posts:

[Speculative Fuction Bot](https://capybasilisk.com/posts/2020/04/speculative-fiction-bot/)

[EXP-RTL: Exponential Retaliation In Iterated Prisoner's Dilemma Games](https://capybasilisk.com/posts/2020/04/exp-rtl-exponential-retaliation-in-iterated-prisoners-dilemma-games/)

[Interleaved Neighborhood Algorithm: Fully Exploratory Optimization](https://capybasilisk.com/posts/2020/06/interleaved-neighborhood-algorithm-fully-exploratory-optimization/)

&nbsp;

[About Me](https://capybasilisk.com/about/)


