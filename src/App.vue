<template>
  <div>
    <h3>SVG</h3>
    <button @click="load">Test</button>
    <div id="mysvg"></div>
  </div>
</template>
<style>
* {
  color:black;
}
.link-label {
}
body {  
  background-color:#eaeaea;
}
#mysvg {
  min-height:300px;
  min-width:500px;
  border:1px solid #cecece;
}
</style>
<script>
import * as d3 from "d3";
import { Runtime } from '@observablehq/runtime';

export default {
  mounted() {
    this.load()
  },
  methods:{
    load(){
      const width = 1200;
      const height = 1200;
      const links = this.links.map((d) => ({ ...d }));
      const nodes = this.nodes.map((d) => ({ ...d }));      

      const simulation = d3.forceSimulation(nodes)
      .force("link",d3.forceLink(links).id((d) => d.id ))
      .force("charge",d3.forceManyBody().strength(-25000))
      .force("center",d3.forceCenter(width/2, height/2))
      .on("tick",ticked);

      const svg = d3.create("svg")
      .attr("width",width)
      .attr("attr",height)
      .attr("viewBox",[0,0,width,height])
      .attr("style","max-width:100%;height:auto;");
      
      const link = svg.append("g")
      .attr("stroke","#eb4034")
      .attr("stroke-opacity",1)
      .selectAll()
      .data(links)      
      .join("line")
      .attr("class","links")
      .attr("stroke-width", d => 1);

      let node_radius = 30;
      const node = svg.append("g")
      .attr("stroke","#eb4034")
      .attr("stroke-width",1)
      .selectAll("circule")
      .data(nodes)
      .enter()
      .append("circle")
      .attr("r",node_radius)
      .attr("fill",d => d.bg)
      
      let node_label = svg.append("g")
      .attr("class","node-labels")
      .selectAll("text")
      .data(nodes)
      .enter()
      .append("text")
      .text(d =>  d.ip )
      .style('text-anchor', 'middle')
      .style('font-size', '12px');

      let target_link_label = svg.append("g")
      .attr("class","link-labels")
      .selectAll("text")
      .data(links)
      .enter()
      .append("text")
      .text((d)=> d.target.id )
      .style("text-anchor","middle")
      .style("font-size","15px");

      let source_link_label = svg.append("g")
      .attr("class","source-link-labels")
      .selectAll("text")
      .data(links)
      .enter()
      .append("text")
      .text((d)=> d.source.id )
      .style('font-weight', 'bold')
      .style("text-anchor","middle")
      .style("font-size","15px");

      
      node.call(
        d3.drag().on("start",dragStarted)
        .on("drag",dragged)
        .on("end",dragEnded)
      );

      function dragStarted(event){
        if(!event.active) {
          simulation.alphaTarget(1).restart();
        }
        console.log(event);
        event.subject.fx = event.subject.x;
        event.subject.fy = event.subject.y;
      }
      function dragEnded(event){
        event.subject.fx = null;
        event.subject.fy = null;
        console.log(event)
      }

      function dragged(event){
        event.subject.fx = event.x;
        event.subject.fy = event.y;
      }
      function dragStarted(event){
        if(!event.active) {
          simulation.alphaTarget(1).restart();
        }
        console.log(event);
        event.subject.fx = event.subject.x;
        event.subject.fy = event.subject.y;
      }

      function getQuadrant(x,y){
        if(x >= 0 && y >= 0){
          // x positive, y positive
          return 1;
        }
        else if(x < 0 && y >= 0){
          // x negative, y positive
          return 2;
        }
        else if(x < 0 && y < 0){
          // x negative, y negative
          return 3;
        }
        else if(x >= 0 && y < 0){
          // x negative, y negative
          return 4;
        }
        return null;
      }
      function flipDegrees(quandrant,degrees){
        if(quandrant == 1){
          return degrees = degrees - 90;
        }
        else if(quandrant == 2){
          return degrees = degrees - 90;
        }
        else if(quandrant == 3){
          return degrees = degrees - 180;
        }
        else if(quandrant == 4){
          return degrees = degrees - 270;
        }
        return degrees;
      }
      

      function ticked(){
        link.attr("x1", d => d.source.x )
        .attr("y1", d => d.source.y )
        .attr("x2", d => d.target.x )
        .attr("y2", d => d.target.y );

        node.attr("cx", d => d.x ).attr("cy", d => d.y );
        node_label.attr("x",d => d.x).attr("y", d => d.y );
        //target_link_label.attr("x",d => d.target.x).attr("y", d => d.target.y);

        target_link_label.attr("transform", d => {

          let delta_x = d.target.x - d.source.x;
          let delta_y = d.target.y - d.source.y;
          let pos_x = d.target.x;
          let pos_y = d.target.y;
          let middle_x = (d.target.x + d.source.x) / 2;
          let middle_y = (d.target.y + d.source.y) / 2;
          let angle = Math.atan2(delta_y,delta_x);
          let degrees = angle * (180 /Math.PI);
          //if(d.source.id == "ROUTER_A" && d.target.id == "ROUTER_E"){
          //  console.log("TARGET==========================================");
          //  console.log(`Source ${ d.source.id}:`, d.source);
          //  console.log(`Target ${ d.target.id}:`, d.target);
          //  console.log(`delta_x`,delta_x);
          //  console.log(`delta_y`,delta_y);
          //  console.log(`D ${d.source.id} -> ${d.target.id}`,degrees);
          //  console.log(`A ${d.source.id} -> ${d.target.id}`,angle);
          //  console.log(`AS ${d.source.id} -> ${d.target.id}`,Math.atan2(d.target.y,d.target.x) );
          //  console.log(`AT ${d.source.id} -> ${d.target.id}`,Math.atan2(d.source.y,d.source.x) );
          //  console.log("=========================================TARGET");
          //}
          if(degrees > 90 || degrees < -90){
            degrees += 180;
          }
          pos_x = d.target.x + (d.target.x - d.source.x) * -0.25;
          pos_y = d.target.y + (d.target.y - d.source.y) * -0.25;
          let style = `translate(${pos_x},${pos_y}),rotate(${degrees})`;
          return style;
        });
        source_link_label.attr("transform", d => {
          let delta_x = d.source.x - d.target.x;
          let delta_y = d.source.y - d.target.y;
          let pos_x = d.source.x;
          let pos_y = d.source.y;
          let middle_x = (d.target.x + d.source.x) /2;
          let middle_y = (d.target.y + d.source.y) /2;
          let angle = Math.atan2(delta_y,delta_x);
          let degrees = angle * (180 /Math.PI);
          if(d.source.id == "ROUTER_A" && d.target.id == "ROUTER_D"){

            console.log("SOURCE==========================================");
            console.log(`Source ${ d.source.id}:`, d.source);
            console.log(`Target ${ d.target.id}:`, d.target);
            console.log(`delta_x`,delta_x);
            console.log(`delta_y`,delta_y);
            console.log(`D ${d.source.id} -> ${d.target.id}`,degrees);
            console.log(`A ${d.source.id} -> ${d.target.id}`,angle);
            console.log(`AS ${d.source.id} -> ${d.target.id}`,Math.atan2(d.target.y,d.target.x) );
            console.log(`AT ${d.source.id} -> ${d.target.id}`,Math.atan2(d.source.y,d.source.x) );
            console.log(`Quadrant ${d.source.id} -> ${d.target.id}`,getQuadrant(delta_x,delta_y));

            console.log("=========================================SOURCE");
          }
            if(degrees > 90 || degrees < -90){
              degrees += 180;
            }
            pos_x = d.source.x + ((d.target.x - d.source.x) * 0.25);
            pos_y = d.source.y + ((d.target.y - d.source.y) * 0.25);
          let style = `translate(${pos_x},${pos_y}),rotate(${degrees})`;
          return style;
        });
      }
      console.log("target_link_label",target_link_label);
      console.log("source_link_label",source_link_label);
      console.log("node",node);      
      window.runtime = new Runtime();
      //invalidation.then(() => simulation.stop());
      //document.getElementById("mysvg").innerHTML = "";
      //simulation.stop();
      document.getElementById("mysvg").append(svg.node());
    }
  },
  data(){
    return {
      "nodes":[
        {
          "id": "ROUTER_A",
          "ip": "192.168.100.1",
          "bg": "#d4d4d4"
        },
        {
          "id": "ROUTER_B",
          "ip": "192.168.100.2",
          "bg": "#32a852"
        },
        {
          "id": "ROUTER_C",
          "ip": "192.168.100.3",
          "bg": "#32a88b"
        },
        {
          "id": "ROUTER_D",
          "ip": "192.168.100.4",
          "bg": "#3232a8"
        },
        {
          "id": "ROUTER_E",
          "ip": "192.168.100.5",
          "bg": "#630425"
        },
        {
          "id": "ROUTER_F",
          "ip": "192.168.100.6",
          "bg": "#a83266"
        },
        {
          "id": "ROUTER_G",
          "ip": "192.168.100.7",
          "bg": "#a83266"
        },
        {
          "id": "ROUTER_H",
          "ip": "192.168.100.8",
          "bg": "#a83266"
        }
      ],
      "links":[
        {
          "source": "ROUTER_A",
          "target": "ROUTER_B",
          "value": 1,
        },
        {
          "source": "ROUTER_A",
          "target": "ROUTER_C",
          "value": 2,
        },
        {
          "source": "ROUTER_A",
          "target": "ROUTER_D",
          "value": 3,
        },
        {
          "source": "ROUTER_A",
          "target": "ROUTER_E",
          "value": 4,
        },
        {
          "source": "ROUTER_A",
          "target": "ROUTER_H",
          "value": 1,
        },
        {
          "source": "ROUTER_E",
          "target": "ROUTER_F",
          "value": 5,
        },
        {
          "source": "ROUTER_E",
          "target": "ROUTER_G",
          "value": 6,
        }
      ],
      "message":"test"
    }
  }
}
</script>
