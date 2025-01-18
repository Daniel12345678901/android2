Automatic Edit Mode

Objective:

    Instead of an edit button for each note, automatically open a page showing the full note with edit capabilities when a note is clicked.

Steps to Implement Automatic Edit Mode

    Modify the Adapter to Handle Click Events:
        Remove the edit button.
        Set up an OnClickListener to handle note item clicks.

    Open Edit Activity on Note Click:
        Pass the selected note to the edit activity.
        Open the edit activity when a note is clicked.

Detailed Implementation
Step 1: Modify the Adapter to Handle Click Events

In your NotesAdapter, remove the edit button and set up an OnClickListener for each note item.
```
package com.example.blocodenotas.adapters

import android.view.LayoutInflater
import android.view.ViewGroup
import androidx.recyclerview.widget.RecyclerView
import com.example.blocodenotas.databinding.NoteItemBinding
import com.example.blocodenotas.models.Note

class NotesAdapter(
    private val onNoteClick: (Note) -> Unit,
    private val onDeleteClick: (Note) -> Unit
) : RecyclerView.Adapter<NotesAdapter.NoteViewHolder>() {

    private val notes = mutableListOf<Note>()

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): NoteViewHolder {
        val binding = NoteItemBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return NoteViewHolder(binding)
    }

    override fun onBindViewHolder(holder: NoteViewHolder, position: Int) {
        holder.bind(notes[position])
    }

    override fun getItemCount(): Int = notes.size

    fun setNotes(newNotes: List<Note>) {
        notes.clear()
        notes.addAll(newNotes)
        notifyDataSetChanged()
    }

    fun addNote(note: Note) {
        notes.add(note)
        notifyItemInserted(notes.size - 1)
    }

    fun updateNote(note: Note) {
        val index = notes.indexOfFirst { it.id == note.id }
        if (index != -1) {
            notes[index] = note
            notifyItemChanged(index)
        }
    }

    fun removeNote(note: Note) {
        val index = notes.indexOfFirst { it.id == note.id }
        if (index != -1) {
            notes.removeAt(index)
            notifyItemRemoved(index)
        }
    }

    inner class NoteViewHolder(private val binding: NoteItemBinding) : RecyclerView.ViewHolder(binding.root) {

        fun bind(note: Note) {
            binding.textViewTitle.text = note.title
            binding.textViewContent.text = note.content

            // Set up click listener to open note in edit mode
            binding.root.setOnClickListener {
                onNoteClick(note)
            }

            // Remove edit button and other unnecessary UI elements
            binding.buttonEdit.visibility = View.GONE
            binding.buttonDelete.setOnClickListener { onDeleteClick(note) }
        }
    }
}
```

Step 2: Open Edit Activity on Note Click

In your MainActivity, handle the note click event to open the edit activity.
```
package com.example.blocodenotas

import android.content.Context
import android.content.Intent
import android.os.Bundle
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
            onDeleteClick = { note -> deleteNote(note) }
        )
        binding.recyclerView.adapter = notesAdapter

        // Load and display notes
        loadNotes()
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
}
```
