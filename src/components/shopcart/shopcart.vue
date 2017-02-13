<template>
  <div>
    <div class="shopcart">
      <div class="content" @click="toggleList">
        <div class="content-left">
          <div class="logo-wrapper">
            <i class="icon-shopping_cart" :class="{'highlight': totalCount>0}"></i>
            <span class="num" v-show="totalCount>0">{{totalCount}}</span>
          </div>
          <span class="price" :class="{'hightlight': totalPrice>0}">￥{{totalPrice}}</span>
          <i class="divider"></i>
          <span class="desc">另需配送费￥{{deliveryPrice}}元</span>
        </div>
        <div class="content-right" @click.stop.prevent="pay">
          <div class="pay" :class="{'checkout': isCheckout}">
            {{payDesc}}
          </div>
        </div>
      </div>
      <div class="ball-container">
        <div v-for="ball in balls">
          <transition name="drop" @before-enter="beforeDrop" @enter="dropping" @after-enter="afterDrop">
            <div class="ball" v-show="ball.show">
              <div class="inner inner-hook"></div>
            </div>
          </transition>
        </div>
      </div>
      <transition name="fold">
        <div class="shopcart-list" v-show="listshow">
          <div class="list-header">
            <h1 class="title">购物车</h1>
            <span class="empty" @click="emptyCart">清空</span>
          </div>
          <div class="list-content" ref="listContent">
            <ul>
              <li class="food border-1px" v-for="food in selectFoods">
                <span class="name">{{food.name}}</span>
                <span class="price">￥{{food.price * food.count}}</span>
                <span class="cartcontrol-wrapper">          
                <cartcontrol :food="food" @add="drop"></cartcontrol>
             </span>
              </li>
            </ul>
          </div>
        </div>
      </transition>
    </div>
    <transition name="fade">
      <div class="list-mask" v-show="listshow" @click="toggleList">
      </div>
    </transition>
  </div>
</template>


<script>
import cartcontrol from 'components/cartcontrol/cartcontrol'
import BScroll from 'better-scroll'

export default {
  props: {
    selectFoods: {
      type: Array,
      default () {
        return []
      }
    },
    deliveryPrice: {
      type: Number
    },
    minPrice: {
      type: Number
    }
  },
  data () {
    return {
      balls: [{
        show: false
      }, {
        show: false
      }, {
        show: false
      }, {
        show: false
      }, {
        show: false
      }],
      dropBall: [],
      listshow: false
    }
  },
  computed: {
    totalPrice () {
      let total = 0
      this.selectFoods.forEach((food) => {
        total += food.price * food.count
      })
      return total
    },
    totalCount () {
      let count = 0
      this.selectFoods.forEach((food) => {
        count += food.count
      })
      return count
    },
    payDesc () {
      if (this.totalPrice === 0) {
        return '￥' + this.minPrice + '元起送'
      } else if (this.totalPrice > 0 && this.totalPrice < this.minPrice) {
        return '还差￥' + (this.minPrice - this.totalPrice) + '元起送'
      } else {
        return '去结算'
      }
    },
    isCheckout () {
      return this.totalPrice >= this.minPrice
    }
  },
  methods: {
    drop (el) {
      console.log(el)
      for (let i = 0; i < this.balls.length; i++) {
        let ball = this.balls[i]
        if (!ball.show) {
          ball.show = true
          ball.el = el
          this.dropBall.push(ball)
          return
        }
      }
    },
    toggleList () {
      if (!this.totalCount) {
        return
      }
      this.listshow = !this.listshow
    },
    emptyCart () {
      this.selectFoods.forEach((food) => {
        food.count = 0
      })
      this.listshow = false
    },
    _initScroll () {
      if (this.listshow) {
        console.log(this.$refs.listContent)
        this.$nextTick(() => {
          if (!this.foodListScroll) {
            this.foodListScroll = new BScroll(this.$refs.listContent, {
              click: true
            })
          } else {
            this.foodListScroll.refresh()
          }
        })
      }
    },
    pay () {
      if (!this.isCheckout) {
        return
      }
      window.alert('请支付' + this.totalPrice + '元')
        // }
    },
    // transitions: {
    //   drop: {
    beforeDrop (el) {
      console.log('beforeDrop')
      let count = this.balls.length
      while (count--) {
        let ball = this.balls[count]
        if (ball.show) {
          let rect = ball.el.getBoundingClientRect()
            // console.log(ball.el.offsetHeight)
          let x = rect.left - 32
          let y = -(window.innerHeight - rect.top - 22)
            // console.log('x ' + x + 'y ' + y)
          el.style.display = ''
            // el.style.webkitTransform = `translate3D(0, $(y)px, 0)`
            // el.style.transform = `translate3D(0, $(y)px, 0)`
          el.style.webkitTransform = 'translate3d(0,' + y + 'px, 0)'
          el.style.transform = 'translate3d(0,' + y + 'px, 0)'
            // el.style.webkitTransform = 'translate3d(' + x + ',' + y + 'px, 0)'
            // el.style.transform = 'translate3d(' + x + 'px,' + y + 'px, 0)'
          let inner = el.getElementsByClassName('inner-hook')[0]
          inner.style.webkitTransform = 'translate3d(' + x + 'px,0,0)'
          inner.style.transform = 'translate3d(' + x + 'px,0,0)'
        }
      }
    },
    dropping (el, done) {
      // 触发浏览器重绘
      /* eslint-disable no-unused-vars */
      console.log('dropping')
      let rf = el.offsetHeight
      this.$nextTick(() => {
        el.style.webkitTransform = 'translate3d(0, 0, 0)'
        el.style.transform = 'translate3d(0, 0, 0)'
        let inner = el.getElementsByClassName('inner-hook')[0]
        inner.style.webkitTransform = 'translate3d(0,0,0)'
        inner.style.transform = 'translate3d(0,0,0)'
        el.addEventListener('transitionend', done)
      })
    },
    afterDrop (el) {
      console.log('afterDrop')
      let ball = this.dropBall.shift()
      if (ball.show) {
        ball.show = false
        el.style.display = 'none'
      }
    }
    // }
  },
  // 编程式实现shop-list打开、关闭的代码
  // fold: {
  //  beforeEnter(el) {
  //    el.style.height = '0'
  //    el.style.display = ''
  //  },
  //  enter(el) {
  //    let rf = el.offsetHeight
  //    let eleHeight = this.selectFoods.length * 49 + 42
  //    this.$nextTick(() => {
  //      el.style.height = eleHeight + 'px'
  //    })
  //  },
  //  leave(el) {
  //    el.style.height = '0'
  //  },
  //  afterLeave(el) {
  //    el.style.display = 'none'
  //  }
  // }
  components: {
    cartcontrol
  },
  // ,
  // events: {
  //   'cart.add' (target) {
  //     this.drop(target)
  //   }
  // },
  watch: {
    'listshow': '_initScroll'
  }
}
</script>



<style lang="stylus" rel="stylesheet/stylus">
@import "../../common/stylus/mixin.styl"

	.shopcart
		position: fixed
		left: 0
		bottom: 0
		z-index: 50
		width: 100%
		height: 48px	
		color: rgba(255, 255, 255, 0.4)
		.content
			display: flex
			background: #141d27
			font-size: 0
			.content-right			
				flex: 0 0 105px
				width: 105px			
				.pay
					height: 48px
					line-height: 48px
					text-align: center				
					font-size: 12px
					font-weight: 700						
					background-color: rgba(255, 255, 255, 0.1)
					&.checkout									
						background-color: rgb(0, 180, 60)
						color: #fff					
			.content-left
				flex: 1
				position: relative			
				.logo-wrapper
					display: inline-block
					vertical-align: top
					position: relative
					top: -10px		
					width: 56px
					height: 56px		
					border-radius: 50%				
					margin: 0 12px
					padding: 6px
					box-sizing: border-box								
					text-align: center
					background: #141d27
					z-index: 10
					.num					
						position: absolute
						top: 0
						right: 0
						width: 24px
						height: 16px
						line-height: 16px					
						font-size: 9px
						font-weight: 700					
						color: #fff
						background-color: rgb(240, 20, 20)
						box-shadow: 0 4px 8px 0 rgba(0,0,0,0.4)
						border-radius: 8px					
					.icon-shopping_cart
						display: inline-block
						width: 100%
						height: 100%
						line-height: 44px
						background-color: rgba(255, 255, 255, 0.1)				
						border-radius: 50%
						font-size: 24px
						&.highlight
							background-color: rgb(0, 160, 220)
							color: #fff						
				.price
					display: inline-block
					vertical-align: top		
					font-size: 16px		
					font-weight: 700
					line-height: 48px
					&.hightlight
						color: #fff				
				.divider
					display: inline-block
					height: 48px
					margin: 0 12px
					color: rgba(255, 255,255,0.1)
					position: relative		
					&:after
						content: ''
						background: rgba(255, 255,255,0.1)			
						position: absolute
						top: 12px
						bottom: 12px
						width: 1px								
				.desc
					display: inline-block
					vertical-align: top		
					font-size: 10px		
					line-height: 48px			
		.ball-container
			.ball
				position: fixed
				left: 32px
				bottom: 32px
				z-index: 200
				&.drop-enter-active, &.drop-leave-active
					transition: all .4s cubic-bezier(0.49, -0.29, 0.75, 0.41)
					.inner
						width: 16px
						height: 16px
						border-radius: 50%
						background: rgb(0, 160, 220)
						transition: all .4s linear
		.shopcart-list
			position: absolute
			left: 0
			top: 0
			z-index: -1
			width: 100%
			background-color: #fff
			width: 100%		
			color: rgb(7, 17, 27)	
			max-height: 240px   
			transform: translate3d(0, -100%, 0)  
			&.fold-enter-active, &.fold-leave-active 
				transition: all .5s			
			&.fold-enter, &.fold-leave-active
				transform: translate3d(0, 100%, 0)
			// max-height: 240px				
			.list-header
				height: 40px
				line-height: 40px			
				padding: 0 18px			
				background-color: #f3f5f7
				border-bottom: 1px solid rgba(7, 17, 27, 0.1)
				.title
					float: left
					font-size: 14px
					color: rgb(7, 17, 27)				
				.empty				
					float: right				
					font-size: 12px
					color: rgb(0, 160, 220)
			.list-content
				overflow: hidden
				padding: 0 18px
				max-height: 217px
				background: #fff			
				.food		
					padding: 12px 0
					box-sizing: border-box
					position: relative
					border-1px(rgba(7, 17, 27, 0.1))
					.name
						line-height: 24px
						font-size: 14px
						color: rgb(7,17,27)
					.price
						position: absolute
						right: 90px
						bottom: 12px					
						line-height: 24px
						font-size: 14px
						font-weight: 700
						color: rgb(240, 20, 20)						
					.cartcontrol-wrapper
						position: absolute					
						right: 0
						bottom: 6px
	.list-mask
		position: fixed
		top:0
		bottom: 0
		left:0
		right: 0
		z-index: 40	
		backdrop-filter: blur(10px)
		opacity: 1
		background-color: rgba(7,17,27,0.6) 
		&.fade-enter-active, &.fade-leave-active
			transition: all .5s		
		&.fade-enter, &.fade-leave-active
			opacity: 0
			background-color: rgba(7,17,27,0)
</style>
