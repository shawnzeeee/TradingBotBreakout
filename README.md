# TradingBotBreakout
namespace NinjaTrader.NinjaScript.Strategies
{
	public class Pullback : Strategy
	{
		int index = 0;
		double upperTest;
		double bottomTest;
		List <double> max = new List <double>();
		List <int> maxIndex = new List<int>();
		List <double> min = new List <double>();
		List <int> minIndex = new List <int>();
	    List <Lines> upperLines = new List <Lines>();
//		List <Lines> bottomLines = new List<Lines>();
		int test1Index, test2Index;
		double direction = 0;
		double localMax;
		double upperResistance;
		double upperSlope, upperIntercept, upperIndex;
		double bottomSlope, bottomIntercept, bottomIndex;
		int maxCount;
		private SMA smaFast;
		private SMA smaSlow;
		bool checkUpperTrend = false;
	    string time;
		int recentMaxIndex;
		double targetProfit;
		double stopLoss;
		protected override void OnStateChange()
		{
			if (State == State.SetDefaults)
			{
				Description									= @"Enter the description for your new custom Strategy here.";
				Name										= "Pullback";
				Calculate									= Calculate.OnBarClose;
				EntriesPerDirection							= 1;
				EntryHandling								= EntryHandling.AllEntries;
				IsExitOnSessionCloseStrategy				= true;
				ExitOnSessionCloseSeconds					= 30;
				IsFillLimitOnTouch							= false;
				MaximumBarsLookBack							= MaximumBarsLookBack.TwoHundredFiftySix;
				OrderFillResolution							= OrderFillResolution.Standard;
				Slippage									= 0;
				StartBehavior								= StartBehavior.WaitUntilFlat;
				TimeInForce									= TimeInForce.Gtc;
				TraceOrders									= false;
				RealtimeErrorHandling						= RealtimeErrorHandling.StopCancelClose;
				StopTargetHandling							= StopTargetHandling.PerEntryExecution;
				BarsRequiredToTrade							= 20;
				// Disable this property for performance gains in Strategy Analyzer optimizations
				// See the Help Guide for additional information
				IsInstantiatedOnEachOptimizationIteration	= true;
			}
			else if (State == State.Configure)
			{
				AddPlot(new Stroke(Brushes.Blue), PlotStyle.Line, "upper"); 
			}
		}

		protected override void OnBarUpdate()
		{				
			//	smaFast = SMA(5);
			  //  smaSlow = SMA(20);
				if(CurrentBar > 1){
						if(Bars.GetHigh(CurrentBar) - Bars.GetHigh(CurrentBar-1) < 0 && direction > 0){
							max.Add(Bars.GetHigh(CurrentBar-1));	
							maxIndex.Add(CurrentBar-1);
							upperLines.Clear();
							if(min.Count > 15){
								min.RemoveAt(0);	
							}
							if(max.Count > 15){
								max.RemoveAt(0);
								upperTrend();
							} 
						}	
						if(Bars.GetLow(CurrentBar) - Bars.GetLow(CurrentBar-1) > 0 && direction < 0){
							min.Add(Bars.GetLow(CurrentBar-1));
							minIndex.Add(CurrentBar-1);
						}
						direction = Bars.GetHigh(CurrentBar) - Bars.GetHigh(CurrentBar-1);
						if(max.Count > 15){
							max.RemoveAt(0);
							maxIndex.RemoveAt(0);
						}
								PrintTo = PrintTo.OutputTab1;
								Print(Bars.GetTime(CurrentBar-1) + ": " +trendState());		
					if(upperLines.Count > 0){
						PrintTo = PrintTo.OutputTab2;
						Print(Bars.GetTime(CurrentBar) +": " + upperLines[upperLines.Count-1].slope + "  " + upperLines[upperLines.Count-1].intercept + "  " + upperLines.Count + "  " + upperLines[upperLines.Count-1].tipCount);
						upperTest = upperLines[upperLines.Count-1].slope*(CurrentBar - upperLines[upperLines.Count-1].index) + upperLines[upperLines.Count-1].intercept;
					//	bottomTest = bottomSlope*(CurrentBar - bottomIndex) + bottomIntercept;
						if(Bars.GetClose(CurrentBar) > upperTest){
						   // stopLoss = Math.Round((upperTest - bottomTest)/0.25); 
							//targetProfit = stopLoss*2;
							SetProfitTarget(CalculationMode.Ticks, 4);
							SetStopLoss(CalculationMode.Ticks , 4);							
							EnterLong(3);
							upperLines.RemoveAt(upperLines.Count-1);
						}
					}
					
				}
				
		  
			
	  }
		protected override void OnMarketData(MarketDataEventArgs marketDataUpdate){
			if(marketDataUpdate.MarketDataType == MarketDataType.Last){
					
			}
		}
		 private bool contractionPhase(){
				 
			 return true;
		 }
		 public class Lines{
			 public double slope;
			 public double intercept ;
			 public double index; 
			 public int tipCount;
			 public Lines(double _slope, double _intercept, double _index, int _tipCount){
					 slope = _slope;
				     intercept = _intercept;
				 	 index = _index;
					 tipCount = _tipCount;
			 }
			 
			
		}
		private double trendState(){
			double sum = 0;
			for(int i = 1; i < max.Count; i++){
				sum += (max[i] - max[i-1])/(maxIndex[i]-maxIndex[i-1]);		
			}
			return sum/15;
		}
		 private void upperTrend(){
		    double test1 = 0,test2 = 0, test3 = 0;
			double testSlope = 0;
			double testIntercept;
			for(int i = 0; i < max.Count-1; i++){
					testSlope = (max[max.Count-1]-max[i])/(maxIndex[max.Count-1]-maxIndex[i]);
					testIntercept = max[max.Count-1];
					int tipCount = 2;
					for(int j = i+1; j <= max.Count-1; j++){
						test1 = testSlope*(maxIndex[j] - maxIndex[max.Count-1]) + testIntercept;
						if(minIndex[j] < maxIndex[j] && minIndex[j] > maxIndex[j-1]){
							if(max[j-1] - min[j] > 3 && max[j] - min[j] > 3){
								if(Math.Abs(max[j] - test1) <= 1){
									tipCount += 1;
								}
							}
						}
					}
						if(tipCount >= 3){
							upperLines.Add(new Lines(testSlope, testIntercept, maxIndex[max.Count-1],tipCount));
						}

			}
		   for(int j = upperLines.Count-1; j >= 0; j--){
				if(upperLines[j].slope > -0.1){
					upperLines.RemoveAt(j);	
				}
		   }
		   int countMax = 0;
		   int countMaxIndex = 0;;
		   for(int i = upperLines.Count-1; i >= 0; i--){
				 if(upperLines[i].tipCount > countMax){
					countMax = upperLines[i].tipCount;
					countMaxIndex = i;
				 }
		   }
		   for(int i = upperLines.Count-1; i >= 0; i--){
				if(i != countMaxIndex) upperLines.RemoveAt(i);   
		   }
		}		
		private void bottomTrend(){
		    double test1 = 0,test2 = 0, test3 = 0;
			double testSlope = 0;
			double testIntercept;	
			int i = min.Count-2;
			for(i = min.Count-2; i >=1; i--){
					testSlope = (min[min.Count-1]-min[i])/(minIndex[min.Count-1]-minIndex[i]);
					testIntercept = min[min.Count-1];
					for(int j = i-1; j >= 0; j--){
						test1 = testSlope*(minIndex[j] - minIndex[min.Count-1]) + testIntercept;
						//if(min[j] - test1 > 3) break;
						if(Math.Abs(min[j] - test1) <= 2){
							//PrintTo = PrintTo.OutputTab2;
							//Print(Bars.GetTime(minIndex[min.Count-1]) + ": " + min[min.Count-1] + ", " + Bars.GetTime(minIndex[i]) + ": " + min[i] + ", " + Bars.GetTime(minIndex[j]) + ": " +  min[j] + ", " + test1);
							bottomSlope = testSlope;
							bottomIntercept = testIntercept;
							bottomIndex = minIndex[min.Count-1];
							break;
						} 
					}
					if(bottomSlope != 0 && bottomIntercept != 0) break;
			}
		}

	}
}


