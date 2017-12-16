---
title: TheReviewIndex | Unbiased User Review Summaries | Monochrome
description: This is an introduction to thereviewindex.com.
header: TheReviewIndex | Unbiased User Review Summaries
comments: true
---

The internet is a source of a humongous amount of unstructured content for all sorts of products, in the form of blogs, videos, user and expert reviews, etc. It is almost impossible for anyone to scour all of this disparate data, with contradictory opinions, and thereafter make a correct, or at least optimal, purchase decision.

{% highlight cpp linenos %}
int getCorresPoint(Point p, Mat& img, int ndisp) {
  ofstream mfile;
  mfile.open("cost.txt");
  int w = 5;
  long minCost = 1e9;
  int chosen_i = 0;
  int x0r = kpr[0].pt.x;
  int y0r = kpr[0].pt.y;
  int ynr = kpr[kpr.size()-1].pt.y;
  int x0l = kpl[0].pt.x;
  int y0l = kpl[0].pt.y;
  int ynl = kpl[kpl.size()-1].pt.y;
  for (int i = p.x-ndisp; i <= p.x; i++) {
    long cost = -1;
    for (int j = -w; j <= w; j++) {
      for (int k = -w; k <= w; k++) {
        if (!isLeftKeyPoint(p.x+j, p.y+k) || !isRightKeyPoint(i+j, p.y+k))
          continue;
        int idxl = (p.x+j-x0l)*(ynl-y0l+1)+(p.y+k-y0l);
        int idxr = (i+j-x0r)*(ynr-y0r+1)+(p.y+k-y0r);
        cost += costF(img_left_desc.row(idxl), img_right_desc.row(idxr));
      }
    }
    cost = cost / ((2*w+1)*(2*w+1));
    mfile << (p.x-i) << " " << cost << endl;
    if (cost < minCost) {
      minCost = cost;
      chosen_i = i;
    }
  }
  cout << "minCost: " << minCost << endl;
  return chosen_i;
}
{% endhighlight %}

Generally, the main problem any average consumer faces while trying to research products online is that there are too many sites with far too much information, and there is typically a disparity in the various opinions expressed. Also, it is not easy to differentiate between what is genuine, and what isn’t.

To help resolve these issues, [The Review Index](https://thereviewindex.com) has launched its discovery and research engine, which simplifies the process of researching products online. It scours various sources on the internet, and mines opinions and reviews for the products. This information is then presented as a unified, unbiased, feature-wise summary scorecard of the aggregate opinion to the end user.

> [The Review Index](https://thereviewindex.com)

Leveraging cutting-edge Machine Learning Algorithms and Artificial Neural Networks, The Review Index compiles reviews from popular sources which are relevant, and focuses on detection of fine-grained topics and sentiment implied therein. It focuses on evaluating the pros and cons as indicated by the reviewers, while ensuring that fraudulent content is identified and removed.

<center>$\mathbf{P} = \mathbf{K}\left[\begin{array}{c|c}
\mathbf{I} & \mathbf{0} \\
\end{array}\right] \quad \quad \mathbf{P'} = \mathbf{K'}\left[\begin{array}{c|c}
\mathbf{R} & \mathbf{t} \\
\end{array}\right]$</center>
<br>

The public beta was launched in January 2017 with six product categories, which include [Mobiles](https://thereviewindex.com/mobiles), [Speakers](https://thereviewindex.com/speakers), [Televisions](https://thereviewindex.com/televisions), [Routers](https://thereviewindex.com/routers), [Microwaves](https://thereviewindex.com/microwaves) and [Washing Machines](https://thereviewindex.com/washingmachines) and caters to Indian consumers. Reviews from popular online stores are compiled and reconciled using Neural Network based algorithms, and feature-wise summaries of the aggregated opinion is created.

Thus, this search engine enables users to discover the optimal products, which satisfy their various requirements. The users are also provided with access to feature-wise ratings, summary, list of pros and cons, as well as user comment snippets. Eventually, the engine compares the prices for the displayed products across all the online stores, to conclude which one has the most cost-effective offering.

The plan is to expand to more categories such as electronics, home appliances, etc. in the near future, while at the same time expanding the sources of opinions.


