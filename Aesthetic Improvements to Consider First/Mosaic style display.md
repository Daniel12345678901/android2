1. Mosaic Style Display

Objective:

    Display notes in a mosaic style with two notes wide starting from the left.

Steps to Implement Mosaic Style Display

    Update the Layout XML:
        Use a RecyclerView with a GridLayoutManager to display items in a grid format.

    Modify the Adapter:
        Ensure the adapter is set up to handle the grid layout.

Detailed Implementation
Step 1: Update the Layout XML

Open your activity_main.xml file and ensure the RecyclerView is properly defined:
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:padding="8dp"/>
</LinearLayout>
```

Step 2: Set Up GridLayoutManager in MainActivity

In your MainActivity, set up the RecyclerView to use a GridLayoutManager:
```
package com.example.blocodenotas

import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.GridLayoutManager
import com.example.blocodenotas.adapters.NotesAdapter
import com.example.blocodenotas.databinding.ActivityMainBinding
import com.example.blocodenotas.models.Note

class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding
    private lateinit var notesAdapter: NotesAdapter

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        // Set up RecyclerView with GridLayoutManager
        binding.recyclerView.layoutManager = GridLayoutManager(this, 2)  // 2 columns in the grid
        notesAdapter = NotesAdapter(
            onEditClick = { note -> editNote(note) },
            onDeleteClick = { note -> deleteNote(note) }
        )
        binding.recyclerView.adapter = notesAdapter

        // Load and display notes
        loadNotes()
    }

    private fun loadNotes() {
        // Logic to load notes from SharedPreferences or other storage
    }

    private fun editNote(note: Note) {
        // Logic to edit note
    }

    private fun deleteNote(note: Note) {
        // Logic to delete note
    }
}
```

Step 3: Modify the Adapter (if needed)

Ensure your NotesAdapter is set up correctly to handle the grid layout. If your adapter is already working with a RecyclerView, you may not need to make any changes here.

If your note item layout (note_item.xml) needs adjustments to look good in a grid, you may want to update its design to fit better in a smaller space.
