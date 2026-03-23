Explanation of our business:
1) We take nfl football cards of players
2) Decide cards for a custom mystery box collections where there are a few expensive cards and a lot of cheap cards
3) Pack those cards in a boxes
4) Use our website to prepare a place for us to record buyers
   1) On website we create a new entity called livestream
   2) Then we create N breaks (basically N collections of boxes) and assign them to the livestream
   3) For each break now we create 32 teams because we sell NFL team claims, not the boxes themselves. So each team claim is a spot in the box, and the buyer of that spot will get the cards in the box assigned to that team.
5) Start livestream where we sell spots into our mystery box collection
    1) When the spot (which is a team claim) is sold, we record the buyer information in our website and assign the buyer to the team claim
6) Once all spots are sold, we open boxes and reveal the cards to the buyers
7) When stream is finished, we do the shipping
   8) Right now we have a Shipping Page where we associate buyers with teams from all breaks they were assigned and gather these cards into a single bundle

What we want to change:
Change everything after bullet 4 to the following:
1) When we decide cards, we also take photos of those cards and upload them to our website storage
2) Use our website to prepare a place for us to record buyers
   1) On website we create a new entity called livestream
   2) Then we create N breaks (basically N collections of boxes) and assign them to the livestream
   3) This time we create M spots because each box can be claimed separately
   4) Then we mark which spots are valuable and those cards from those boxes will be displayed in the OBS display
3) Use OBS to display photos of all available valuable cards in the current collection in a rectangular area. It's important for them to be shuffled and resized in such way that whole area is filled with them 
4) Start livestream where we sell next box in our mystery box collection
5) When the box is sold, we open the box and reveal the cards to the buyer.
   1) Also we assign the buyer to the box they just bought and that we were opening on the livestream
   2) If this box was marked as a box with a valuable card, that this card would be automatically removed from OBS display
6) Sell the next box in our mystery box collection and repeat steps 4-5 until all boxes are sold.
7) When stream is finished, we do the shipping
   8) Somehow we need to associate buyers with all cards they won and make that in an obvious failure-proof way. Just images? 


What we need to plan:
1) How to make photos of those cards
2) How to upload them to our website storage
3) How to create a dedicated page in our website that will display those photos in a rectangular area and shuffle them every time a box is sold
   1) We will use this page in our OBS
4) How to re-shuffle cards with minimum changes on dedicated page each time a valueable card is sold and removed from the display

Let's introduce terms:
1) Our website will be called "System"
2) Website page where we create livestreams is "Livestream Page"
3) Website page where we create breaks and assign them to livestreams is "Breaks Page"
4) Website page where we create spots and assign them to breaks is "Spots Page"
3) Website page where we display valuable cards is "Cards Board Page"

References:
1) Frontend code - https://github.com/SaloEater/Whatnot-Frontend
2) Backend code - https://github.com/SaloEater/WhatNot-Webhook-Holder
