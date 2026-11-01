# The drop reel

**[Open the drop reel](../drops/index.html)**, a full-width interactive page
with every simulated drop: its ParaView movie, its numbers, its time series
and its nine analysis figures.

What you can do there:

- **Organize** the drops by bed type, by start conditions (arrival speed and
  angle), by drop set or by outcome, as cards with movies or as a sortable
  table. The filters on the left narrow everything at once and show how many
  drops each choice leaves.
- **Overview**: a matrix of bed and craft against start conditions, each cell
  the median [speed ratio](../glossary.md#speed-ratio-e) with its outcomes
  (click a cell to see those drops); an explorer that plots any two numbers
  against each other (click a point to open the drop); and a map of the
  landing spots seen from above.
- **Open a drop**: the movie plays with charts of height, speed, spin,
  contact force and energy that follow it (a playhead tracks the movie;
  click a chart to move the movie there), the craft's path seen from above,
  the verdict on its last contact, every number of its record, and its
  figures.
- **Every number has an ⓘ** that opens its definition, why it can be
  trusted, and the code that computes it. Dotted words open the glossary.

The reel is rebuilt with `python3 lab.py site --reel <drop sets> --publish`
when drop sets finish or are re-simulated. The movies and data live on the
documentation repository's `drops-media` branch, so they never bloat its
history.
