# 타입스크립트 정규분포 정규화

## 1. 표준편차 구하기
[Calculating Standard Deviation with Array.map and Array.reduce, In JavaScript](https://derickbailey.com/2014/09/21/calculating-standard-deviation-with-array-map-and-array-reduce-in-javascript/)
 
## 2. 정규분포값으로 백분위 구하기 

  1. gaussian 라이브러리 사용. Licence: none
  2. gaussian 라이브러리의 npm 링크: https://www.npmjs.com/package/gaussian
  3. gaussian.cdf(Normal Cumulative Distribution Function, 누적 분포 함수) 메소드 사용으로 정규분포값의 백분위를 얻을 수 있다.
 
## 3. NormalDistribution 유틸리티 클래스 제작

```typescript
// Client
let data = [0.1, 0.3, 0.5, 0.7, 1, 1.5, 1.6, 2.0];
let normalDistribution = new NormalDistribution(data);

data.forEach(d=> {
  console.log(d + ": " + normalDistribution.normalProbability(d));
});

/*
0.1: 0.08762127697535108
0.3: 0.14888692442593462
0.5: 0.23364700571243455
0.7: 0.33996502869053236
1: 0.5234988718459825
1.5: 0.8008783400385715
1.6: 0.841810783856089
2: 0.9485114169289138
*/
```

```typescript
// NormalDistribution 유틸리티 클래스

declare const gaussian: any; 

export class NormalDistribution {

  private gaus: any;
  private avg: number;
  private stde: number;

  constructor(private values: number[]) {
    this.gaus = gaussian(0, 1);
    this.avg = this.average(values);
    this.stde = this.standardDeviation(this.avg, values);
  }

  normalProbability(value: number): number {
    return this.gaus.cdf(this.normalDistributionValue(this.avg, this.stde, value));
  }

  private normalDistributionValue(average: number, standardDeviation: number, value: number): number {
    return (value-average)/standardDeviation;
  }

  private standardDeviation(average: number,values: number[]): number{
    let avg = average;

    let squareDiffs = values.map(function(value){
      let diff = value - avg;
      let sqrDiff = diff * diff;
      return sqrDiff;
    });

    let avgSquareDiff = this.average(squareDiffs);

    let stdDev = Math.sqrt(avgSquareDiff);
    return stdDev;
  }

  private average(data: number[]): number{
    let sum = data.reduce(function(sum, value){
      return sum + value;
    }, 0);

    let avg = sum / data.length;
    return avg;
  }
}
```