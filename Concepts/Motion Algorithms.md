# Motion Algorithms

A big focus for this year has been motion algorithms for autonomous. Motion algorithms allow for a faster, easier, and more competitive autonomous. This year, we have built an **X-Drive** robot, which opens up a wide range of possibilities. Having a holonomic chassis means that we can have *heading-agnostic* movement functions, meaning that we can drive anywhere while turning at a completely independent angle.

However, to be able to have a consistent autonomous while not moving in linear or straight manners, we need robot localisation. To do this, we used the odometry algorithm provided by team [5225A](https://www.vexforum.com/t/team-5225-introduction-to-position-tracking-document/49640) from ontario. 