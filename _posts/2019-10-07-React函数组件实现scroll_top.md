--- 
layout:     post
title:      React函数组件实现Scroll Top
subtitle:   
date:       2019-10-07
author:     wells
header-img: img/c2.jpg
catalog: true
tags:
    - React 
--- 



## React函数组件实现Scroll Top

```react
import React,{ useState, useEffect} from 'react'
import { ScrollTop } from './style'

//函数组件
const Example = () => {
    
    //初始化scrollTop的state值为false
    const [scrollTop,setScrollTop] = useState(false)
    
    //hook钩子监听滚动事件
    useEffect(() => {
		window.addEventListener("scroll",changeScrollTopShow())
    	return () => {
     	window.addEventListener("scroll",changeScrollTopShow())
    	}
  	}); 
    
    //判断scrollTop距离并更新state
  	const changeScrollTopShow = () => {
    	if(document.documentElement.scrollTop > 350){
      		setScrollTop(true)
    	}else{
      		setScrollTop(false)
    	}
	}
    
    //点击TOP按钮
    const handleGoTop = () => {
        //window.scrollTo(0,0)  //无动画版

        //动画版
        let scrollToTop = window.setInterval(function() {
            let pos = window.pageYOffset;
            if ( pos > 0 ) {
                window.scrollTo( 0, pos - 20 ); // 每一步滚多远
            } else {
                window.clearInterval( scrollToTop );
            }
         }, 2); 
     }
    
    
    return(
    	<div>
            //state状态为true时显示组件
        	{scrollShow &&
          		<ScrollTop onClick={ handleGoTop }>TOP</ScrollTop>
        	}
        </div>
    )
}


```



