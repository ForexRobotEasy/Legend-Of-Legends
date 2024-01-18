//+------------------------------------------------------------------+
//|                    Legend Of Legends                             |
//|                Developer's Site: forexroboteasy.com             |
//|               Development by: Forex Robot Easy Team              |
//+------------------------------------------------------------------+

//--- Include necessary libraries
#include <Trade\Trade.mqh>
#include <Renko\Renko.mqh>

//--- Global variables
CTrade trade; // instance of CTrade class

//--- Indicator input parameters
input int indicatorPeriod = 96; // number of candlesticks for indicator calculation

//--- Expert Advisor input parameters
input bool useGridStrategy = true; // enable grid strategy
input bool useMartingaleStrategy = true; // enable martingale strategy
input int gridStep = 10; // grid step size
input double lotSize = 0.01; // initial lot size
input double martingaleMultiplier = 2.0; // multiplier for martingale strategy

//--- Indicator objects
CRenko renko; // instance of CRenko class
double renkoHigh[]; // array to store renko high values
double renkoLow[]; // array to store renko low values

//+------------------------------------------------------------------+
//| Expert Advisor initialization function                           |
//+------------------------------------------------------------------+
int OnInit()
{
   //--- Initialize indicator
   renko.Create(IndicatorShortName(), indicatorPeriod);
   
   //--- Initialize indicator arrays
   ArrayResize(renkoHigh, indicatorPeriod);
   ArrayResize(renkoLow, indicatorPeriod);
   
   //--- Set default settings for EA
   trade.SetExpertMagicNumber(Symbol(), 123456); // set magic number
   trade.SetSlippage(10); // set slippage
   
   //--- Set default settings for grid and martingale strategy
   if(useGridStrategy)
   {
      trade.SetDeviationInPoints(50); // set deviation for pending orders
      trade.SetUseStopLoss(true); // enable stop loss
   }
   
   if(useMartingaleStrategy)
   {
      trade.SetMaxLots(10); // set maximum lot size for martingale
   }
   
   //--- Return initialization result
   return(INIT_SUCCEEDED);
}

//+------------------------------------------------------------------+
//| Expert Advisor start function                                     |
//+------------------------------------------------------------------+
void OnTick()
{
   //--- Check if indicator is calculated
   if(renko.IndicatorCounted() < indicatorPeriod)
      return;
   
   //--- Get indicator values
   ArrayCopy(renko.High(), renkoHigh, 0, indicatorPeriod);
   ArrayCopy(renko.Low(), renkoLow, 0, indicatorPeriod);
   
   //--- Perform market analysis and generate trading signals
   for(int i = 0; i < indicatorPeriod; i++)
   {
      //--- Enter the market within the channel if conditions are met
      if(renkoHigh[i] > renkoHigh[i+1] && renkoLow[i] < renkoLow[i+1])
      {
         //--- Enter Buy trade
         if(useGridStrategy)
            trade.BuyGrid(Symbol(), lotSize, renkoLow[i], gridStep);
         else
            trade.Buy(Symbol(), lotSize, renkoLow[i], 0, 0);
         
         //--- Increase lot size for martingale strategy
         if(useMartingaleStrategy)
            lotSize *= martingaleMultiplier;
      }
      
      //--- Enter the market within the channel if conditions are met
      if(renkoLow[i] < renkoLow[i+1] && renkoHigh[i] > renkoHigh[i+1])
      {
         //--- Enter Sell trade
         if(useGridStrategy)
            trade.SellGrid(Symbol(), lotSize, renkoHigh[i], gridStep);
         else
            trade.Sell(Symbol(), lotSize, renkoHigh[i], 0, 0);
         
         //--- Increase lot size for martingale strategy
         if(useMartingaleStrategy)
            lotSize *= martingaleMultiplier;
      }
   }
}

//+------------------------------------------------------------------+
//| Expert Advisor deinitialization function                         |
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
{
   //--- Close any open trades
   trade.CloseAll();
}

/*******************************************************
 * PRODUCT DESCRIPTION
 *******************************************************/

// This Expert Advisor (EA) is called 'Legend Of Legends' and is developed by Forex Robot Easy Team. It is designed to trade based on Renko chart patterns and uses a combination of grid and martingale strategies.

// The EA utilizes the Renko indicator to calculate the high and low values of the Renko bars. It analyzes the market within the channel formed by the Renko bars and generates trading signals based on the conditions met.

// The input parameters allow customization of the EA's trading strategies. The 'useGridStrategy' parameter enables or disables the grid strategy, which involves placing pending orders at fixed intervals. The 'gridStep' parameter determines the distance between the pending orders.

// The 'useMartingaleStrategy' parameter enables or disables the martingale strategy, which involves increasing the lot size after each losing trade. The 'martingaleMultiplier' parameter determines the multiplier for the lot size.

// The EA's initialization function sets default settings for the EA, such as the magic number and slippage. It also sets default settings for the grid and martingale strategies if enabled.

// In the start function, the EA checks if the Renko indicator is calculated and then retrieves the indicator values. It performs market analysis and generates trading signals based on the conditions met within the Renko channel. It enters buy or sell trades based on the grid strategy or regular trading strategy, and adjusts the lot size if the martingale strategy is enabled.

// The deinitialization function closes any open trades before the EA is terminated.

// For detailed reviews and trading results of this product, please visit the Forex Robot Easy website: https://forexroboteasy.com/forex-robot-review/legend-of-legends-ea-in-depth-forex-software-review/

// Note: ForexRobotEasy is not the official developer of this product. This code is provided as a sample that demonstrates how the product works. To find the official developer of this product, please refer to MQL5.
