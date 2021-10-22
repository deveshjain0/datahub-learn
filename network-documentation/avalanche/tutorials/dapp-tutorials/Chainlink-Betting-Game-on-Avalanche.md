# Chainlink Betting Game on Avalanche

#Introduction 
This is a blockchain based betting game where  you can bet on the outcome of a dice roll with cryptocurrency and if you guessed right then you  double your money this game is powered by ethereum smart contacts thats run on the blockchain and we’re going to use the chainlink protocol to implement randomness for out dice roll. 







#Begain with

 Here is the application will work when the user will connect to their web browser with metamask they’ll talk to a front-end application built in react.js and then application will talk directly to the ethereum blockchain and on the blockchain we’ll create a smart contract that implements the betting game and that’s going to use the chainlink protocol which of course talked to the chainlink smart contacts. So the user flow is here is that they make a bet to directly to our smart contacts with the funded application ,if they guess the number right thet will win twice the amount of cryptocurrency that they bet.


# requirements 
Node.js 
Chainlink
Metamask 





#how to use chainlink 

Chainlink is an external data provider for the blockchain,so its an oracle service which means that it provides real-world data to smart contacts. 

In this chainlink is use to provide cryptocurrency prices to smart contracts like exchanges
 We focus on today is chainlink ability to provide randomness to smart contracts which is an essential feature for gaming there are some inherent 

# node.js 
 After installation go to your terminal typing 

node -v 


# metamask 

Google metamask web extension

# download github to terminal 

Open Terminal then type 

Git clone “link” 


After downloading. Then, type in terminal 

Cd chainlink_betting_game 

For install all dependencies for the projects with npm 

In terminal.          

Npm  install 


Once all your dependencies install go ahead and open up a project text editor 


All the code is going to be inside the source directory (src).   Contact directory for the smart contracts that we’ll put on the ethereum blockchain 

So bettingame.sol.  Is going to be the main smart contract that we’re going to use for this tutorial 

Components directory is for the application code written,this is the main app component like navbar.js and main.js and app.js 



Open file explorer application.   Copy the bettingGame.sol code and paste in the application. Then compile first , use 0.66.  Because we’re using a solidity version for this particular tutorial ,then compile it . 
Make sure everything works properly 
Next we want to deploy it and put it on the blockchain so we’re using the test network for this . 
We’re going to use the rinkeby test network, in order to do that 



In metamask. You have to make sure that you’re connected to the rinkeby test network. 


#main game function 

Look  at out chart so basically the user makes a bet directly to to our smart contract by calling the game function and what they do is they bet on a dice roll and so they bet the low value or the high value which is going to be either one through three or three to six . They provide a random seed for that number and if they win twice the amount of cryptocurrency that they bet. And if not then they lose the cryptocurrency.   



Open terminal. 

Cd chainlink_betting_game 

Then   Type 

Npm run start 

This will run the web server

Once its done you can see the application loaded in your browser here . This should be automatically popup. 

You can see your account that you’re connected with here in the top right hand corner 
Betting game application on top left hand corner 
And here it is our little dice game and we can play the game ,well first we have got the max bet that’s the exact amount of ethereum cryptocurrency thar we send to the smart contract. And the balance is your current wallet balance of your account which is connected to the metamask 
 
Lets bet the 0.5 ethereum, then lets playing…….




