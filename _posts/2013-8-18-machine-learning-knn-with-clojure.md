---
layout: post
title: 机器学习笔记 -- k-临近算法
---

k-临近算法是一种分类算法, 计算输入值与各个特征值之间距离来进行分类.

简单的概括一下算法的思路就是: 

1. 算出输入值与各个特征值之间的距离, 按升序排序. 
1. 获取前k个分类
1. 选择出现次数最高的分类, 就是输入值所属的分类.

k-临近算法优缺点分析

* 优点: 精度高, 对异常值不敏感, 无数据输入假定.
* 缺点: 计算复杂度高, 空间复杂度高.
* 适用数据范围: **数值型** 和 **标称型**

以下通过clojure来实现kNN算法:

    (defn create-test-data []  ; 生成测试数据
      '(((0.1 0.1)
         (0.0 0.0)
		 (0.2 0.2)
		 (1.1 1.1)
		 (1.0 1.0))
		(:a :a :a :b :b)))
    
    (defn distance [xs ys]  ; 计算两个向量之间的距离
	  (. java.lang.Math sqrt (apply + (map #(* % %) (map #(- %1 %2) xs ys)))))
    
    (defn count-list-element [xs]  ; 计算列表中每个元素的出现次数
      (reduce
       (fn [cnts x]
        (let [n (x cnts)]
         (if (= n nil)
          (assoc cnts x 1)
          (assoc cnts x (+ n 1)))))
	   {}
       xs))
    
    (defn kNN [inX dataSet labels k]
     (let* [label-groups (map #(concat [%1] %2) labels dataSet)  ; 将分类与特征值合并, 方便后续处理
            label-groups-sort-by-distance (sort-by  ; 计算输入值与特征值的距离并且排序
                                           #(distance inX (rest %1)) label-groups)
            k-labels (take k (map first label-groups-sort-by-distance))  ; 获取前K个分类
            labels-group-by-count (sort-by first (count-list-element k-labels))]  ; 计算每个分类的出现次数
           (-> labels-group-by-count  ; 返回出现次数最多的分类
               reverse
               first
               first)))

    (defn -main [& args]
	  (let [[groups labels] (create-dataset)]
        (println (kNN [0.0 0.0] groups labels 3))))

附带上我的[机器学习笔记](https://github.com/PeerXu/machine-learning-clojure)

感谢阅读
