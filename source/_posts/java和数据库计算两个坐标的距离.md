title: "java和数据库计算两个坐标的距离"
date: 2015-04-21 00:41
category: Java
tags: [地图, 坐标运算, 距离]

---

##使用Java进行计算

```java
/**
 * 类功能说明：地图坐标距离计算工具类
 * Title: DistanceUtil.java
 * @author liuxing
 * @date 2013-9-8 下午10:36:03
 * @version V1.0
 */
public class DistanceUtil {

	private static double DEF_PI = Math.PI; // PI
	private static double DEF_2PI = Math.PI * 2; // 2*PI
	private static double DEF_PI180 = Math.PI / 180; // PI/180.0
	private static double DEF_R = 6370693.5; // 地球半径

	/**
	 * 
	 * 功能说明：计算两个地图坐标点之间的距离（近距离计算）
	 * liuxing 2013-9-8 下午10:42:17
	 * @param lng1 起点经度
	 * @param lat1 起点纬度
	 * @param lng2 终点经度
	 * @param lat2 终点纬度
	 * @return
	 */
	public static double getShortDistance(double lng1, double lat1, double lng2, double lat2) {
		double ew1, ns1, ew2, ns2;
		double dx, dy, dew;
		double distance;
		// 角度转换为弧度
		ew1 = Math.toRadians(lng1);
		ns1 = Math.toRadians(lat1);
		ew2 = Math.toRadians(lng2);
		ns2 = Math.toRadians(lat2);
		
		// 经度差
		dew = ew1 - ew2;
		// 若跨东经和西经180 度，进行调整
		if (dew > DEF_PI){
			dew = DEF_2PI - dew;
		} else if (dew < -DEF_PI){
			dew = DEF_2PI + dew;
		}
			
		dx = DEF_R * Math.cos(ns1) * dew; 	// 东西方向长度(在纬度圈上的投影长度)
		dy = DEF_R * (ns1 - ns2); 			// 南北方向长度(在经度圈上的投影长度)
		// 勾股定理求斜边长
		distance = Math.sqrt(dx * dx + dy * dy);
		return distance;
	}

	/**
	 * 
	 * 功能说明：计算两个地图坐标点之间的距离（远距离计算）
	 * liuxing 2013-9-8 下午10:43:21
	 * @param lng1 起点经度
	 * @param lat1 起点纬度
	 * @param lng2 终点经度
	 * @param lat2 终点纬度
	 * @return
	 */
	public static double getLongDistance(double lng1, double lat1, double lng2, double lat2) {
		double ew1, ns1, ew2, ns2;
		double distance;
		
		// 角度转换为弧度
		ew1 = lng1 * DEF_PI180;
		ns1 = lat1 * DEF_PI180;
		ew2 = lng2 * DEF_PI180;
		ns2 = lat2 * DEF_PI180;
		
		// 求大圆劣弧与球心所夹的角(弧度)
		distance = Math.sin(ns1) * Math.sin(ns2) + Math.cos(ns1) * Math.cos(ns2) * Math.cos(ew1 - ew2);
		// 调整到[-1..1]范围内，避免溢出
		if (distance > 1.0){
			distance = 1.0;
		} else if (distance < -1.0){
			distance = -1.0;
		}
		
		// 求大圆劣弧长度
		distance = DEF_R * Math.acos(distance);
		return distance;
	}
	
	public static void main(String[] args) {
		double mLat1 = 31.24081800000000; 	// point1纬度
		double mLng1 = 121.46541700000000; 	// point1经度
		double mLat2 = 31.239946;	// point2纬度
		double mLng2 = 121.466417;	// point2经度
		
		double distanceByShort = getShortDistance(mLng1, mLat1, mLng2, mLat2);
		System.out.println(distanceByShort);
		
		double distanceByLong = getLongDistance(mLng1, mLat1, mLng2, mLat2);
		System.out.println(distanceByLong);
	}

}
```

##使用SqlServer函数计算

其他数据库版本请找到相应的函数替换后移植

```sql
-- =============================================
-- Author:      liuxing
-- Create date: 2013-09-10
-- Description:	计算2个坐标点的距离（短距离计算）
-- =============================================
CREATE function dbo.fn_getShortDistance(
	 @lng1 decimal(19,11)
	,@lat1 decimal(19,11)
	,@lng2 decimal(19,11)
	,@lat2 decimal(19,11)
)
returns decimal(19,11)
AS
BEGIN
	--declare @lng1 decimal(19,11)
	--declare @lat1 decimal(19,11)
	--declare @lng2 decimal(19,11)
	--declare @lat2 decimal(19,11)


	--set @lat1 = 31.238662--; 	// point1纬度
	--set @lng1 = 121.466633--; // point1经度
	--set @lat2 = 31.239727--;	// point2纬度
	--set @lng2 = 121.462745--;	// point2经度
	declare @ew1 decimal(19,11)
		, @ns1 decimal(19,11)
		, @ew2 decimal(19,11)
		, @ns2 decimal(19,11)
		, @dx decimal(19,11)
		, @dy decimal(19,11)
		, @dew decimal(19,11)
		, @distance decimal(19,11)
	-- 角度转换为弧度
	set @ew1 = Radians(@lng1)-- * 0.01745329252;
	set @ns1 = Radians(@lat1)-- * 0.01745329252;
	set @ew2 = Radians(@lng2)-- * 0.01745329252;
	set @ns2 = Radians(@lat2)-- * 0.01745329252;
	-- 经度差
	set @dew = @ew1 - @ew2;
	-- 若跨东经和西经180 度，进行调整
	if (@dew > Pi())
	begin
		set @dew = 2 * Pi() - @dew;
	end
	else if (@dew < -Pi())
	begin
		set @dew = 2 * Pi() + @dew;
	end
		
	set @dx = 6370693.5 * Cos(@ns1) * @dew -- 东西方向长度(在纬度圈上的投影长度)
	set @dy = 6370693.5 * (@ns1 - @ns2)   -- 南北方向长度(在经度圈上的投影长度)
	-- 勾股定理求斜边长,开平方根
	set @distance = sqrt(@dx * @dx + @dy * @dy);
	return @distance;
	--print @distance
END
```