# Open Decisions

| Question | Decision |
|---|---|
| How quickly should the board reflect a card being marked as sold? | Doesn't matter — viewers don't see the board at the moment a card is being marked |
| What feedback should the operator see after marking a card sold? | Card changes its visual status and re-sorts to the sold section; no extra signal needed |
| Should there be a separate Sold panel? | No — operator page shows all cards (unsold first, sold after) in one sorted grid |
| How long should local scan folders be kept after upload? | Operator deletes them manually; no automatic cleanup |
| When the last row is short, are cards stretched proportionally on both axes or only horizontally? | TBD |
| How is a zoomed card de-zoomed? | Either click the same card again or click anywhere outside it |
| Can multiple cards be zoomed simultaneously? | No — one at a time; clicking another card swaps the zoom to it |
| Is 175% a fixed scale or configurable? | Fixed at 175% |
| Does the zoom overlay the surrounding cards, or does the layout reflow? | Overlay — zoomed card scales up on top with higher z-index, grid does not reflow |
