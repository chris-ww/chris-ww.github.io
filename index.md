---
layout: single
---



- [New York City Crime Dashboards/Analysis](/projects/crime/){: .btn .btn--primary} [**Github Code**](https://github.com/chris-ww/crime-data){: .btn .btn--primary}
    - **Description:** Using Reported crime data from New york City
    - **Tools:** R, Shiny  
    - **Tags:** Big Data, Dashboards, Data analysis  
    
- [Gathering and Analyzing Personal Data](/projects/tracking/){: .btn .btn--primary} [**Github code**](https://github.com/chris-ww/personal-data){: .btn .btn--primary}
    - **Description:** Use API's to gather and transform data from exercise tracker, calendar and location history.  
    - **Tools:** Python, R, airflow, SQlite, html, json, csv, db  
    - **Tags:** Data engineering, API's, analytics  


- [Scheduling](/projects/scheduler/){: .btn .btn--primary} [**Github Code**](https://github.com/chris-ww/scheduler){: .btn .btn--primary}
    - **Description:** A simple scheduling package to assign/reassign employees to tasks
    - **Tools:** Python, MySQL, Cron
    - **Tags:** scheduling
    

<div id="box">
    <div id="mydiv">
       <img src="img/key.png" alt="key">
    </div>
</div>
<html>
<style>
#mydiv {
  position: absolute;
  cursor: move;
  z-index: 9999999999999999999999999;
}

</style>



<script>



function isCollide(a, b) {
    var aRect = a.getBoundingClientRect();
    var bRect = b.getBoundingClientRect();
    return !(
        ((aRect.top + aRect.height) < (bRect.top)) ||
        (aRect.top > (bRect.top + bRect.height)) ||
        ((aRect.left + aRect.width) < bRect.left) ||
        (aRect.left > (bRect.left + bRect.width))
    );
}

//Make the DIV element draggagle:
dragElement(document.getElementById("mydiv"));

function dragElement(elmnt) {
  var pos1 = 0, pos2 = 0, pos3 = 0, pos4 = 0;
  if (document.getElementById(elmnt.id)) {
    document.getElementById(elmnt.id).onmousedown = dragMouseDown;
  } else {
    elmnt.onmousedown = dragMouseDown;
  }

  function dragMouseDown(e) {
    e = e || window.event;
    e.preventDefault();
    // get the mouse cursor position at startup:
    pos3 = e.clientX;
    pos4 = e.clientY;
    document.onmouseup = closeDragElement;
    // call a function whenever the cursor moves:
    document.onmousemove = elementDrag;
  }

  function elementDrag(e) {
    e = e || window.event;
    e.preventDefault();
    // calculate the new cursor position:
    pos1 = pos3 - e.clientX;
    pos2 = pos4 - e.clientY;
    pos3 = e.clientX;
    pos4 = e.clientY;
    // set the element's new position:
    elmnt.style.top = (elmnt.offsetTop - pos2) + "px";
    elmnt.style.left = (elmnt.offsetLeft - pos1) + "px";
  }

  function closeDragElement() {
    /* stop moving when mouse button is released:*/
    document.onmouseup = null;
    document.onmousemove = null;
    if(isCollide(document.getElementById("mydiv"),document.getElementById("lock"))){
        document.getElementById("p1").innerHTML ="You have unlocked a useless trophy!!";
        document.getElementById("lock").src = '/img/trophy.png';
    }
  }
}
</script>

</html>
