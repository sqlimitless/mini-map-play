<script setup lang="ts">
import { read, utils, writeFile } from 'xlsx';

const XLSX = require("xlsx");
// XLSX is a global from the standalone script
let drawer:any;
async function handleFileAsync(e:any) {
  const file = e.target.files[0];
  const data = await file.arrayBuffer();
  /* data is an ArrayBuffer */
  const workbook = XLSX.read(data);

  /* DO SOMETHING WITH workbook HERE */
  const rowJson = utils.sheet_to_json<any>(workbook.Sheets[workbook.SheetNames[0]]);

  const jsonData:any = {};
  rowJson.forEach((row:any) => {
    Object.keys(rowJson[0]).forEach((header:string,index: number) => {
        if(!jsonData[header]) jsonData[header] = [];
        jsonData[header].push(row[header]);
    });
  })

  drawer = '';
  const balls:any = [];
  Object.keys(jsonData).forEach((v:string) => {
    const ball = new Ball(v);
    ball.addRouteArr(jsonData[v]);
    balls.push(ball);
  })
  drawer = new Drawer(balls);
}

function move() {
    drawer.move();
}

class Route{
    private transform:string;
    private x:number;
    private y:number;
    constructor(x:number,y:number){
      this.x = x;
      this.y = y;
      this.transform = `translate(${x}px, ${y}px)`;
    }
    public getTransform():string{
        return this.transform;
    }
    getXY():{x:string,y:string}{
      return {x: this.x.toString(),y: this.y.toString()}
    }
}

class Ball {
    
    private ballRoute: Array<Route> = new Array();
    private ballName:string;

    constructor(ballName:string){
        this.ballName = ballName;
    }

    addRoute = (route:Route) => {
        if(route.getTransform() == undefined || !route.getTransform().includes('translate')){
            throw new Error(`transform의 값이 translate가 아님. \n route.transform : ${route.getTransform()}`)
        }
        this.ballRoute.push(route);
    }

    addRouteArr = (route:Array<string>) => {
        route.forEach((v:string) => {
            const vArr = v.split(',')
            this.addRoute(new Route(vArr.at(0),vArr.at(1)))
        })
    }

    getBallName = ():string => {
        return this.ballName;
    }

    getBallRoute = ():Array<Keyframe> => {
        const arr = new Array<Keyframe>();
        this.ballRoute.forEach(element => {
            arr.push({'transform':element.getTransform()})
        });
        return arr;
    }

    getFirstPosition = ():{x:string,y:string} => {
      return this.ballRoute[0].getXY();
    }
}

class Drawer {
    
    private balls:Array<Ball>;
    constructor(balls:Array<Ball>){
        this.balls = balls;
        this.makeBall();
    }

    private makeBall = () => {
        /**
         * balls 갯수만큼 공 모양 DIV 생성 (각 공에 고유ID부여, 공 이름 노출)
         * #area에 append
         */
        this.balls.forEach((ball:Ball,index:number) => {
            const divParent = document.createElement('div');
            const divName = document.createElement('div');
            const divElem = document.createElement('div');
            divElem.className = 'ball';
            const backColor:string = `#${Math.floor(Math.random() * 16777215).toString(16)}`;
            divElem.style.cssText = `float:left; position:relative; width:10px; height: 10px; border-radius: 50%; border: 1px solid;  background-color:${backColor}`
            divParent.setAttribute('id',`ball_${index}`);
            divParent.style.position = 'absolute';
            divParent.style.display = 'none';
            divName.innerText = ball.getBallName();
            divName.style.cssText = `position: relative; float:left; color:${backColor}`
            divParent.appendChild(divElem)
            divParent.appendChild(divName)
            document.querySelector('#area')?.appendChild(divParent);
        })
    }

    move = () => {
        /**
         * 각 ball을 경로대로 움직이게 animate 부여
         * 옵션값 한 좌표당 2초, 끝나고 마지막 상태 유지
         */

         this.balls.forEach((ball:Ball,index:number) => {
            const animationOption = {
              duration: 2000 * ball.getBallRoute().length,
              fill: 'forwards'
            }
            const ballDiv = document.querySelector(`#ball_${index}`);
            ballDiv?.animate(ball.getBallRoute(),animationOption);
            ballDiv.style.display = 'block';
        })
    }

    clear = () => {
        /**
         * 모든 정보 삭제
         */
    }
}
</script>
<template>
    <input type="file" @change="handleFileAsync($event)">
    <input type="button" @click="move" value="MOVE">
    <div id="area"></div>
</template>
<style scoped>
.ball{
    position:relative;
    width:10px;
    height: 10px;
    border-radius: 50%;
    border: 1px solid;
}
</style>