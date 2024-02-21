---
title: calculateTax
categories:
  - 默认
excerpt: '计算年度个人所得税（2023）'
description: '计算年度个人所得税（2023）'
date: 2024-02-21 07:16:33
---

## 纳税标准及计算公式

- 计算公式1: `应纳税所得额 = 年度总收入（含劳务报酬）- (各项专项附加扣除 + 起征线5000) * 工作月数`
![专项扣除](/image/calculateTax/2023.jpg)

- 计算公式2: `纳税金额 = 应纳税所得额 * 各阶段税率`
1. 年度不超过36000元的税率为：3 % 
2. 超过36000 - 144000元的部分税率为：10 % 
3. 超过144000 - 300000元的部分税率为：20 % 
4. 超过300000 - 420000元的部分税率为：25 % 
5. 超过420000 - 660000元的部分税率为：30 %
6. 超过660000 - 960000元的部分税率为：35 %
7. 超过960000元的税率为：45 %


## 计算函数


```js
/**
 * @param {*} total 年度总收入
 * @param {*} { child: 子女教育, mortgage: 贷款, rentHouse: 租房, support: 赡养老人 }
 * @param {*} other: 其他扣除，比如大病、再教育等
 * @param {*} workMonths: 工作月数
 */
// 既要还房贷又要付租金，个税专项附加能同时扣除吗？
// 纳税人在主要工作城市没有自有住房而发生的住房租金支出；纳税人的配偶在纳税人的主要工作城市有自有住房的，视同纳税人在主要工作城市有自有住房。
function calculator(
  total,
  { child = 0, mortgage = 0, rentHouse = 0, support = 0 } = {},
  other = 0,
  workMonths = 12,
) {
  const deduct = (child + mortgage + rentHouse + support) * workMonths + other; // 扣除部分
  let needPayTaxes = total - (deduct + workMonths * 5000); // 应缴税部分

  const taxRates = [
    { maxIncome: 36000, taxRate: 0.03 },
    { maxIncome: 144000, taxRate: 0.1 },
    { maxIncome: 300000, taxRate: 0.2 },
    { maxIncome: 420000, taxRate: 0.25 },
    { maxIncome: 660000, taxRate: 0.3 },
    { maxIncome: 960000, taxRate: 0.35 },
    { maxIncome: Infinity, taxRate: 0.45 },
  ];

  let tax = 0;

  for (const rate of taxRates) {
    if (needPayTaxes <= rate.maxIncome) {
      tax += needPayTaxes * rate.taxRate;
      break;
    }
    tax += rate.maxIncome * rate.taxRate;
    needPayTaxes -= rate.maxIncome;
  }

  tax = Math.round(tax);
  console.log(`应缴税：${tax}`);
  return tax;
}
```

调用函数:
> 年收入 20588，租房 1500、赡养老人 1500，再教育 3600，工作10个月

```js
calculator(20588, { rentHouse: 1500, support: 1500 }, 3600, 10)
```