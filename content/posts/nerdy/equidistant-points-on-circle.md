+++
tags = [
  "tech", "math"
]
summary = "How to find the positions of a number of equidistant points on a circle, including C++ code to do it for you."
logo = ""
showLogo = false
draft = false
date = "2017-03-29T11:34:07+02:00"
title = "Finding equidistant points on a circle"
hasMath = true
+++

We know that every circle has \\(360\degree = 2\pi\\). It's what makes them round.   
How can we find the `(x,y)` positions of `n` that are evenly distributed on a circle?

*Trigonometry* to the rescue! All points *on* a circle are equidistant to the circle's center. All we need to do is go around the circle evenly so that we arrive where we started after `n` steps.

For this, we put each point at an angle \\(\theta\\) that goes from `360/n` to `360` evenly in `n` steps:

$$\theta = \frac{1}{n} \cdot 360,\;\; \frac{2}{n} \cdot 360,\;\; \ldots,\;\; \frac{n}{n} \cdot 360$$

Then we must know that a point `(x,y)` at an angle \\(\theta\\) is located at

$$(x,y) = (x\_0 + r \cdot cos(\theta), y\_0 + r \cdot sin(\theta))$$

where \\(x\_0, y\_0\\) are the center of the circle and \\(r\\) is the circle's radius.

So calculate \\((x,y)\\) for all \\(\theta\\) and you're all set!   
For simplicity's sake, here is some `C++` code that does exactly this and also computes pairwise distances:

```c++
#include <iostream>
#include <stdlib.h>
#include <math.h>
#include <vector>

using namespace std;

class Coord {
  public:
    Coord(double x, double y) : x(x), y(y) {}
    double x, y;

    double distance(const Coord& other) const {
      return sqrt(pow(x - other.x, 2) + pow(y - other.y, 2));
    }

};

int main(int argc, char** argv) {
  if (argc < 5) {
    cerr << "I need parameters: <circle center x> <circle center y> <radius> <number of points>" << endl;
    exit(-1);
  }
  double  radius = atof(argv[1]),
          centerX = atof(argv[2]),
          centerY = atof(argv[3]),
          numPoints = atof(argv[4]);
  cout << "Plotting " << numPoints << " equidistant points on a circle with center (x=" << centerX << ", y=" << centerY
       << ") with radius r=" << radius << ":" << endl;

  vector<Coord> coords;
  for (double i = 0; i < numPoints; i++) {
    double angle = (2 * M_PI) * ((i+1) / numPoints); // For all points the angles sum to 360 degrees = 2Pi.
    double x = centerX + radius * cos(angle);
    double y = centerY + radius * sin(angle);
    coords.push_back(Coord(x, y));
  }

  // Print them.
  cout << "P\tx\ty" << endl;
  for (size_t i = 0; i < coords.size(); i++) {
    cout << (i+1) << "\t" << coords.at(i).x << "\t" << coords.at(i).y << endl;
  }

  // Print pairwise distances.
  for (size_t i = 0; i < coords.size(); i++) {
    for (size_t j = 0; j < coords.size(); j++) {
      cout << "d(" << (i+1) << ", " << (j+1) << ") = " << coords.at(i).distance(coords.at(j)) << endl;
    }
  }
  return 0;
}
```

For `n=6, x0=800, y0=300, r=200` we find:

```
P	x	y
1	900	473.205
2	700	473.205
3	600	300
4	700	126.795
5	900	126.795
6	1000	300
d(1, 1) = 0
d(1, 2) = 200
d(1, 3) = 346.41
d(1, 4) = 400
d(1, 5) = 346.41
d(1, 6) = 200
d(2, 1) = 200
d(2, 2) = 0
d(2, 3) = 200
d(2, 4) = 346.41
d(2, 5) = 400
d(2, 6) = 346.41
d(3, 1) = 346.41
d(3, 2) = 200
d(3, 3) = 0
d(3, 4) = 200
d(3, 5) = 346.41
d(3, 6) = 400
d(4, 1) = 400
d(4, 2) = 346.41
d(4, 3) = 200
d(4, 4) = 0
d(4, 5) = 200
d(4, 6) = 346.41
d(5, 1) = 346.41
d(5, 2) = 400
d(5, 3) = 346.41
d(5, 4) = 200
d(5, 5) = 0
d(5, 6) = 200
d(6, 1) = 200
d(6, 2) = 346.41
d(6, 3) = 400
d(6, 4) = 346.41
d(6, 5) = 200
d(6, 6) = 0
```
