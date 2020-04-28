---
layout: ecsoya
title: Drawing Gradient in Draw2d
category: Eclipse
---

一个在Draw2d Figure上画渐进色的示例：

	Figure figure = new Figure();
	figure.setLayoutManager(new XYLayout());

	Figure f = new Figure() {
		protected void paintFigure(Graphics graphics) {
			super.paintFigure(graphics);

			Rectangle rect = getBounds();

			// From white to red vertically.
			graphics.setForegroundColor(ColorConstants.white);
			graphics.setBackgroundColor(ColorConstants.red);
			graphics.fillGradient(new Rectangle(rect.x, rect.y, rect.width,
					rect.height / 2), true);

			// From white to lightGreen horizontally.
			graphics.setForegroundColor(ColorConstants.white);
			graphics.setBackgroundColor(ColorConstants.lightGreen);
			graphics.fillGradient(new Rectangle(rect.x, rect.y
					+ rect.height / 2, rect.width, rect.height / 2), false);
		}

	};
	f.setBorder(new LineBorder(ColorConstants.gray));
	figure.add(f, new Rectangle(10, 10, 100, 100));
  
![]({{site.baseurl}}/images/2015-07-13-gef-1.gif)  