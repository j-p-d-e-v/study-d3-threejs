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
      .force("charge",d3.forceManyBody().strength(-20000))
      .force("center",d3.forceCenter(width/2, height/2))
      .force("x",d3.forceX())
      .force("y",d3.forceY())
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
      .attr("stroke-width", 1);

      let node_radius = 40;
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
      .style('font-size', '11px');

      let target_link_label = svg.append("g")
      .attr("class","link-labels")
      .selectAll("text")
      .data(links)
      .enter()
      .append("text")
      .text((d)=> d.target.id )
      .style("text-anchor","end")
      .style("font-size","11px");

      let source_link_label = svg.append("g")
      .attr("class","source-link-labels")
      .selectAll("text")
      .data(links)
      .enter()
      .append("text")
      .text((d)=> d.source.id)
      .style("fill","red")
      .style('font-weight', 'bold')
      .style("text-anchor","start")
      .style("font-size","11px");

      
      node.call(
        d3.drag().on("start",dragStarted)
        .on("drag",dragged)
        .on("end",dragEnded)
      );

      function dragEnded(event){
        if(!event.active) {
          //Stops the Animation / Simulation
          simulation.alphaTarget(0).restart();
        }
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
          //Starts the Animation/Simulation
          simulation.alphaTarget(1).restart();
        }
        console.log(event);
        event.subject.fx = event.subject.x;
        event.subject.fy = event.subject.y;
      }

      function getDegrees(x1,y1,x2,y2){
          let degrees = getAngle(x1,y1,x2,y2) * (180 /Math.PI);
          return degrees
      }
      
      function getAngle(x1,y1,x2,y2){
          let delta_x = x1 - x2;
          let delta_y = y1 - y2;
          return Math.atan2(delta_y,delta_x);
      }
      function inverseRotate(degrees){
        if(degrees > 90 || degrees < -90){
          return degrees + 180;
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

        target_link_label.attr("transform", d => {
          let degrees = getDegrees(d.target.x,d.target.y,d.source.x,d.source.y);
          //degrees = inverseRotate(degrees);
          let pos_x = d.target.x + (Math.cos(getAngle(d.source.x,d.source.y,d.target.x,d.target.y)) * ((node_radius + 5)));
          let pos_y = d.target.y + (Math.sin(getAngle(d.source.x,d.source.y,d.target.x,d.target.y)) * ((node_radius + 5)));          
          let style = `translate(${pos_x},${pos_y}),rotate(${degrees})`;
          return style;
        });
        source_link_label.attr("transform", d => {
          let degrees = getDegrees(d.source.x,d.source.y,d.target.x,d.target.y);
          //console.log(`SOURCE: ${ d.source.id} -> ${ d.target.id}==========================================`);
          //console.log(`Source: ${d.source.x}, ${d.source.y}`);
          //console.log(`Target: ${d.target.x}, ${d.target.y}`);
          //console.log(`Degrees`,degrees);
          //console.log("=========================================SOURCE");
            if(d.source.id == "ROUTER_A12345433333333333" && d.target.id == "ROUTER_D12345433333333333"){

            }
          //degrees = inverseRotate(degrees);
          let pos_x = d.source.x + (Math.cos(getAngle(d.target.x,d.target.y,d.source.x,d.source.y)) * ((node_radius * 4.30 )+ d.source.id.length ));
          let pos_y = d.source.y + (Math.sin(getAngle(d.target.x,d.target.y,d.source.x,d.source.y)) * ((node_radius * 4.30 )+ d.source.id.length ));
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
      //document.getElementById("mysvg").append(svg.node());
      console.log(svg.node())
    }
  },
  data(){
    return {
      "nodes":[
        {
          "id": "ROUTER_A12345433333333333",
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
          "id": "ROUTER_D12345433333333333",
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
          "source": "ROUTER_A12345433333333333",
          "target": "ROUTER_B",
          "value": 1,
        },
        {
          "source": "ROUTER_A12345433333333333",
          "target": "ROUTER_C",
          "value": 2,
        },
        {
          "source": "ROUTER_A12345433333333333",
          "target": "ROUTER_D12345433333333333",
          "value": 3,
        },
        {
          "source": "ROUTER_A12345433333333333",
          "target": "ROUTER_E",
          "value": 4,
        },
        {
          "source": "ROUTER_A12345433333333333",
          "target": "ROUTER_H",
          "value": 1,
        },
        {
          "source": "ROUTER_A12345433333333333",
          "target": "ROUTER_F",
          "value": 5,
        },
        {
          "source": "ROUTER_A12345433333333333",
          "target": "ROUTER_G",
          "value": 6,
        }
      ],
      "message":"test"
    }
  }
}
</script>
