
import React, { useEffect, useState } from 'react'
import API from './api/api'
import MenuList from './components/MenuList'
import Cart from './components/Cart'
import OrderTracker from './components/OrderTracker'
import AdminDashboard from './components/AdminDashboard'
import io from 'socket.io-client'

const socket = io(import.meta.env.VITE_API_WS || 'http://localhost:5000');

export default function App(){
  const [meals,setMeals] = useState([])
  const [cart,setCart] = useState([])
  const [orders,setOrders] = useState([])

  useEffect(()=>{ API.get('/meals').then(r=>setMeals(r.data)).catch(console.error) },[])

  useEffect(()=>{
    socket.on('order-updated', updated => {
      // update local orders if any
      setOrders(prev => prev.map(o => o._id === updated._id ? updated : o))
    })
    socket.on('new-order', o => console.log('new-order', o))
    return ()=>{ socket.off('order-updated'); socket.off('new-order') }
  },[])

  return (
    <div className="app">
      <header><h1>Campus Smart Eats</h1></header>
      <main>
        <MenuList meals={meals} addToCart={(m)=>setCart(c=>[...c,{...m, qty:1}])} />
        <Cart cart={cart} setCart={setCart} setOrders={setOrders} />
        <OrderTracker orders={orders} />
        <AdminDashboard />
      </main>
    </div>
  )
}
