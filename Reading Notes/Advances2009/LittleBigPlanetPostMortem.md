# Little Big plent post mortem

The generally idea of the talk is to take visual inconsistent and unrelated ideas to piece together a coherent design. Regardles of if the pieces of tech makes sense together

Only do work when you need to, "the fastest computation is the one that you don't do"
95% of geo was was procedurally generated, the 5% remaing was essentially just sack boy so precomputing wasn't a avenue to go down

Post precompution the idea is to reduce spatial frequency of computation, by pushing slolwy varying quantities away from per pixel and into vertex work. Simplfy everything down to one specific code path if possible.

using the spus for vertex transformationsL
