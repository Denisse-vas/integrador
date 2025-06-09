# Integrador
## activity_main 
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:padding="16dp"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <EditText
        android:id="@+id/etProducto"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Nombre del producto" />

    <EditText
        android:id="@+id/etCantidad"
        android:hint="Cantidad"
        android:inputType="number"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <Button
        android:id="@+id/btnAgregar"
        android:text="Agregar Pedido"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <ListView
        android:id="@+id/listaPedidos"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</LinearLayout>

## MainActiviy

package com.example.myapplication

import android.os.Bundle
import android.widget.*
import androidx.appcompat.app.AppCompatActivity
import com.google.firebase.database.*

data class Pedido(val id: String = "", val producto: String = "", val cantidad: Int = 0)

class MainActivity : AppCompatActivity() {
    private lateinit var database: DatabaseReference
    private lateinit var adapter: ArrayAdapter<String>
    private val listaTextos = mutableListOf<String>()
    private val listaIds = mutableListOf<String>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val etProducto = findViewById<EditText>(R.id.etProducto)
        val etCantidad = findViewById<EditText>(R.id.etCantidad)
        val btnAgregar = findViewById<Button>(R.id.btnAgregar)
        val listaPedidos = findViewById<ListView>(R.id.listaPedidos)

        database = FirebaseDatabase.getInstance().reference.child("pedidos")
        adapter = ArrayAdapter(this, android.R.layout.simple_list_item_1, listaTextos)
        listaPedidos.adapter = adapter

        btnAgregar.setOnClickListener {
            val producto = etProducto.text.toString()
            val cantidad = etCantidad.text.toString().toIntOrNull() ?: 0
            val id = database.push().key ?: return@setOnClickListener
            val pedido = Pedido(id, producto, cantidad)
            database.child(id).setValue(pedido)
            etProducto.text.clear()
            etCantidad.text.clear()
        }

        database.addValueEventListener(object : ValueEventListener {
            override fun onDataChange(snapshot: DataSnapshot) {
                listaTextos.clear()
                listaIds.clear()
                for (child in snapshot.children) {
                    val pedido = child.getValue(Pedido::class.java)
                    pedido?.let {
                        listaTextos.add("${it.producto} (${it.cantidad})")
                        listaIds.add(it.id)
                    }
                }
                adapter.notifyDataSetChanged()
            }

            override fun onCancelled(error: DatabaseError) {}
        })

        listaPedidos.setOnItemLongClickListener { _, _, position, _ ->
            val id = listaIds[position]
            database.child(id).removeValue()
            true
        }
    }
}
