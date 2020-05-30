---
layout: post
title: How a Nerd Buys a House
subtitle: A Mortgage Net Present Value Calculator
image: assets/img/money-house.jpg
gh-repo: swheaton/mortgage-present-value-calc
gh-badge: [star, fork, follow]
tags: [python, personal project]
ext-js: https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML
---

## Mortgage Decision Woes
I bought my house almost 2 years ago now, and one of the big lessons I learned is that there's always another lesson to be learned.  Totally :corn:, but it's true.  I first smacked into this once we finally chose a house and had our offer accepted.  I'm done with spreadsheets, pro/con lists, and cost analyses, right? **WRONG**! The process of paying for the house presented almost as many questions and decision points as finding the house. How does a coder deal with this? Python of course.

## Mortgage 101
Let's start out slow. Skip [ahead](#eval) if you know the basics.

### Variables
There are a number of variables to consider with a mortgage, that either vary by loan or chosen home. These include enough quantity and quality to facilitate discussion in the rest of this post, but by no means serve as a complete picture of the mortgage-related terms that will pop up.

#### Loan Variables
Interest Rate
: The percentage of the loan amount that will be owed as interest per year.

Points
: One "point" is one percentage point of the loan amount, which can be paid upfront in exchange for "buying down" a lower interest rate.

PMI
: Private Mortgage Insurance - insurance you may be required to pay for, which gives the lender money if you default on the mortgage loan. Read: you buy this _for the lender_ and get no benefit out of it whatsoever. Until your equity in the house exceeds 20% of the loan amount, PMI is mandatory. The lender may offer to pay this monthly cost, but it will cost you points or a higher interest rate.

Loan Term
: The number of years you have to pay back the loan.  Typically 15 or 30 years, the latter being most common.
{:.dl-horizontal}

#### Home Variables
Principal
: The amount of money loaned by the lender; equal to the home price minus down payment.

Down Payment
: The amount of money paid upfront towards the loan, not including closing costs. Paying a larger down payment shows lenders greater financial means and accountability, leading to lower interest rates and access to a wider range of loan products. It's possible to pay 0% down, but typically 3% of the house price is the minimum. 20% is the amount you've probably heard before because it's what banks like - that's the minimum to get out of paying private mortgage insurance (PMI).

Closing Costs
: Additional fees that are paid at the time of sale, in addition to the down payment. For the buyer this typically includes taxes and fees going to local/state governments and title companies, among other things. Traditionally the real estate agents are paid by the seller.

Escrow Payments
: Monthly payments towards property taxes, home insurance, HOA fees, and other official liabilities that are facilitated by the lender. Often this means you don't have to worry about paying these directly, but as always it's not about you! There's a large backlog of these payments built up into an "escrow" account to make sure that even if you default on the loan, the government still gets their tax money!

Loan Type
: Mortgage loans come in many different flavors but we'll focus on fixed interest rate loans here, where the interest rate stays the same throughout the loan term. Variable rate loans allow the interest rate to change with the going rate of the market (for both good and bad), and so is obviously both more complicated and harder to simulate.
{:.dl-horizontal}

### Monthly Payment
A mortgage loan accrues simple interest, and does not compound. This is because the amount paid is pre-calculated over the life of the loan in what's called the amortization schedule. At the beginning, most of your monthly payment will go towards interest and not paying down the principal. Over time, this flips as the principal goes down and so interest also goes down.  But the monthly payment is always the same, and can be calculated with this equation:

| P | Principal |
| I | Interest Rate (Monthly) |
| N | Loan Term (Months) |
| M | Monthly Payment for Interest and Principal |

$$
M=P*I\frac{(1+I)^N}{(1+I)^N-1}
$$

{: eval}
## Algorithmic Evaluation
So how can we evaluate different loans given all these different variables? Is a .5% interest rate cut worth paying a point up front? Should I pay 10% down and deal with PMI for a few years? How does this change if I want to sell the house at 5 years vs. 10 years? To answer these, I decided to take a page from the business class book and look at net present value.

### Net Present Value (NPV)
The premise of NPV is that paying money a year from now is better - to a calculable degree - than paying that same amount of money now; in an ideal world, you'd invest the sum in the market and keep the earned interest when you have to pay out in a year.  Inflation also plays a role, too, as $100 next year will very likely be worth (relatively) slightly less than $100 today. Inflation and the market average market rage can be combined using the [Fisher Equation](https://en.wikipedia.org/wiki/Fisher_equation), then exponentially compounded monthly to get a discount factor.  By reducing the cost of the year-from-now payment by the discount factor, we get its value in _today's_ dollars, or **Present Value (PV)**. By summing up all of the PVs for all payments and incomes, we arrive at the **Net Present Value (NPV)**. Assuming a simple discount factor of 1.12 per year (1.02 per month), the PV of the $100 payment, discounted monthly, would be **-$78.85**, meaning that paying $100 a year from now is functionally equivalent to paying only **$78.85** today.

$$\begin{eqnarray}
  PV & = & \frac{-100}{1.02^{12}} \\
  & = & -78.85
\end{eqnarray}$$

### NPV Applied
Applying this concept to a mortgage loan, we can calculate the NPV of each loan to be able to directly compare them. While technically you own equity in the house beginning from the down payment, it shouldn't be counted as a positive asset until the house is actually sold. For every year, we can simulate what the NPV would be if we were to sell the house at the end of that year. The table below shows the various PV-related events that can happen in a given month.

|  Month &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | Present-Value Event |
| :------ |:--- |
| Month 0 | Add down payment and closing costs as a negative |
| Month 1-N | Add discounted monthly payment as a negative |
| Month X | PMI is removed from total payment (if applicable) once equity reaches 20% |
| Month N | "Sell" the house and add inflated equity, discounted, as a positive |

### How a Nerd Buys a House
So the answer to 'how a nerd buys a house' is write a Python script! Check it out on the linked [GitHub](https://github.com/swheaton/mortgage-present-value-calc) repo - it's super easy to get running and the parameters are self-descriptive. It only works for fixed-rate mortgages right now; I tried modeling more complicated loan types, but it proved to be not particularly useful given the large assumptions required. Below are the sample parameters provided and the corresponding output. With a target year of year 10, the loans are presented in order of greatest (least negative) NPV given a sale at year 10. All other years are listed as well in tabular form.

### Loans
1. _Loan1-minDown_: A 4.5% interest rate, 30-year term loan where we make the minimum (typically) down payment of 3%. No points and $200 of PMI per month.

2. _Loan1-maxDown_: The same loan except we pay 20% down payment, which allows us to forego PMI payments.

3. _Loan2_: The same loan except we pay down 1 point to earn a 0.5% cut in interest rate - down to 4.0%.

#### Sample Parameters
```json
{
    "market": {
        "avgInflation": 2.0,
        "marketInt": 7.0,
        "agentRate": 6.0
    },
    "houseDetails":
    {
        "price": 322000,
        "annualPropTax": 3500,
        "annualHoaFee": 90,
        "annualInsurance": 900,
        "targetYear": 10
    },
    "loans":
    [
        {
            "name": "Loan1-minDown",
            "type": "fixed",
            "intRate": 4.5,
            "points": 0.0,
            "downPayment": 3.0,
            "pmi": 200,
            "closingCosts": 10000,
            "term": 30
        },
        {
            "name": "Loan1-maxDown",
            "type": "fixed",
            "intRate": 4.5,
            "points": 0.0,
            "downPayment": 20.0,
            "pmi": 0,
            "closingCosts": 10000,
            "term": 30
        },
        {
            "name": "Loan2",
            "type": "fixed",
            "intRate": 4.0,
            "points": 1.0,
            "downPayment": 3.0,
            "pmi": 200,
            "closingCosts": 10000,
            "term": 30
        }
    ]
}
```

#### Sample Output

	New monthly payment (no PMI) for Loan1-minDown at month 109 :1956.75
	New monthly payment (no PMI) for Loan2 at month 103 :1865.33
	=== Initial Monthly Payments ===
	  Loan1-maxDown    Loan2    Loan1-minDown
	---------------  -------  ---------------
	        1679.39  2065.33          2156.75

	=== Net Present Value ===
	      Loan1-maxDown      Loan2    Loan1-minDown
	--  ---------------  ---------  ---------------
	 0         -23783.4   -78863           -24123
	 1         -51270.2   -52037.8         -53085.8
	 2         -67561.2   -68770.9         -70699.2
	 3         -82457.7   -84116.1         -86777.6
	 4         -96087.4   -98195.3        -101461
	 5        -108567    -111119          -114878
	 6        -120001    -122990          -127144
	 7        -130487    -133898          -138364
	 8        -140110    -143929          -148634
	 9        -148410    -153159          -158041
	10        -155531    -161658          -165656
	11        -162092    -169489          -172647
	12        -168146    -176711          -179071
	13        -173738    -183375          -184979
	14        -178910    -189532          -190419
	15        -183701    -195223          -195434
	16        -188145    -200490          -200063
	17        -192274    -205369          -204341
	18        -196115    -209892          -208301
	19        -199695    -214091          -211970
	20        -203037    -217992          -215376
	21        -206161    -221621          -218542
	22        -209087    -225002          -221490
	23        -211832    -228154          -224240
	24        -214412    -231096          -226809
	25        -216841    -233847          -229212
	26        -219131    -236422          -231466
	27        -221294    -238836          -233582
	28        -223341    -241101          -235574
	29        -225282    -243229          -237451
	30        -227123    -245232          -239223

#### Discussion
In this example, **Loan1-maxDown** provides the best NPV at year 10 due to a much lower monthly cost, despite the higher upfront investment in down payment. Because the points payment is sunk at the outset when it's the most costly (in NPV terms), **Loan2** never ends up coming out ahead of the others.  This goes to show why many people think that points are rarely worth it, especially if you don't plan to stick out the term of the loan or never refinance. I found this tool to be incredibly useful when comparing multiple mortgages with varying terms and parameters, and I hope you do too.

----------------

#### Assumptions
- The housing market will not deviate from the average inflation. Obviously not a good assumption since housing depends on location and other factors, but unless one can forecast the sale price of the house, simply applying inflation to the current price is the most reasonable course of action.  

- The average inflation and market growth rate will be constant. Again obviously this cannot be the case, but it's a reasonable assumption given that the future is unknowable.

- You will not get out of PMI early (if applicable) through an assessed home price increase. This is possible but most people won't do it.

- Default market parameters:
	- Default inflation value of 2%. In the US, this is the stated inflation target, so it's a fair assumption.
	- Default market growth rate of 7%, which is considered the average (minus inflation) in modern times.
	- Default agent fee of 6% (buyer and seller 3% each), paid at the time of sale. It's a scam for sure, but unless you go rogue, you're stuck with it.

------------------
_Disclaimer: I am not a lawyer, financial advisor, or mortgage expert, just a lowly software developer who learned some things through the home-buying process. Use this tool at your own risk._
