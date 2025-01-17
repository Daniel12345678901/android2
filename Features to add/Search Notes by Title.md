Allow users to search for notes by their title.

Search Notes by Title

Objective:

    Allow users to search for notes by their title.

Steps to Implement Search Notes by Title

    Add a SearchView to the Layout:
        Update activity_main.xml to include a SearchView.

    Implement Search Functionality in MainActivity:
        Set up a listener to handle text changes in the SearchView.
        Filter the notes displayed in the RecyclerView based on the search query.

Detailed Implementation
Step 1: Add a SearchView to the Layout

Open your activity_main.xml file and add a SearchView above the RecyclerView:
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <SearchView
        android:id="@+id/searchView"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:queryHint="Search notes by title"/>

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:padding="8dp"/>
</LinearLayout>
```

Step 2: Implement Search Functionality in MainActivity

In your MainActivity, set up a listener for the SearchView to filter notes based on the search query.
```
package com.example.blocodenotas

import android.content.Context
import android.content.Intent
import android.os.Bundle
import android.widget.SearchView
import androidx.activity.result.contract.ActivityResultContracts
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.GridLayoutManager
import com.example.blocodenotas.adapters.NotesAdapter
import com.example.blocodenotas.databinding.ActivityMainBinding
import com.example.blocodenotas.models.Note
import com.google.gson.Gson
import com.google.gson.reflect.TypeToken

class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding
    private lateinit var notesAdapter: NotesAdapter
    private val gson = Gson()
    private var notesList: MutableList<Note> = mutableListOf()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        // Set up RecyclerView with GridLayoutManager
        binding.recyclerView.layoutManager = GridLayoutManager(this, 2)  // 2 columns in the grid
        notesAdapter = NotesAdapter(
            onNoteClick = { note -> openEditNoteActivity(note) },
            onDeleteClick = { note -> deleteNote(note) },
            onNoteLongClick = { note -> toggleSelectionMode(note) }
        )
        binding.recyclerView.adapter = notesAdapter

        // Load and display notes
        loadNotes()

        // Set up SearchView listener
        binding.searchView.setOnQueryTextListener(object : SearchView.OnQueryTextListener {
            override fun onQueryTextSubmit(query: String?): Boolean {
                return false
            }

            override fun onQueryTextChange(newText: String?): Boolean {
                filterNotes(newText)
                return true
            }
        })
    }

    private fun filterNotes(query: String?) {
        val filteredList = if (query.isNullOrEmpty()) {
            notesList
        } else {
            notesList.filter { it.title.contains(query, ignoreCase = true) }
        }
        notesAdapter.setNotes(filteredList)
    }

    private val addNoteLauncher = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result ->
        if (result.resultCode == RESULT_OK) {
            val note: Note? = result.data?.getParcelableExtra("note")
            note?.let {
                notesAdapter.addNote(it)
                saveNotes()
            }
        }
    }

    private val editNoteLauncher = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result ->
        if (result.resultCode == RESULT_OK) {
            val note: Note? = result.data?.getParcelableExtra("note")
            note?.let {
                notesAdapter.updateNote(it)
                saveNotes()
            }
        }
    }

    private fun openEditNoteActivity(note: Note) {
        val intent = Intent(this, AddEditNoteActivity::class.java).apply {
            putExtra("note", note)
        }
        editNoteLauncher.launch(intent)
    }

    private fun deleteNote(note: Note) {
        notesAdapter.removeNote(note)
        saveNotes()
    }

    private fun loadNotes() {
        val sharedPreferences = getSharedPreferences("notes_app", Context.MODE_PRIVATE)
        val notesJson = sharedPreferences.getString("notes", "[]")
        val type = object : TypeToken<List<Note>>() {}.type
        notesList = gson.fromJson(notesJson, type)
        notesAdapter.setNotes(notesList)
    }

    private fun saveNotes() {
        val sharedPreferences = getSharedPreferences("notes_app", Context.MODE_PRIVATE)
        val editor = sharedPreferences.edit()
        val notesJson = gson.toJson(notesAdapter.getNotes())
        editor.putString("notes", notesJson)
        editor.apply()
    }

    private fun toggleSelectionMode(note: Note) {
        // Existing selection mode logic
    }

    private fun deleteSelectedNotes() {
        // Existing delete selected notes logic
    }
}
```
