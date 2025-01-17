Hold to Delete

Objective:

    Instead of a delete button for each note, make it so that when you press and hold a note, it will show a blank circle on the top right corner of each note where the user can press and it will show a check mark if it's selected. When at least one note is selected, a delete button will appear on the top right corner.

Steps to Implement Hold to Delete

    Modify the Adapter to Handle Long Press Events:
        Set up a LongClickListener to handle long press events on note items.
        Toggle selection mode when a note is long-pressed.

    Show Selection Circles and Delete Button:
        Display a blank circle on the top right corner of each note when in selection mode.
        Show a delete button when at least one note is selected.

Detailed Implementation
Step 1: Modify the Adapter to Handle Long Press Events

In your NotesAdapter, set up a LongClickListener to handle long press events on note items.
```
package com.example.blocodenotas.adapters

import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import androidx.recyclerview.widget.RecyclerView
import com.example.blocodenotas.databinding.NoteItemBinding
import com.example.blocodenotas.models.Note

class NotesAdapter(
    private val onNoteClick: (Note) -> Unit,
    private val onDeleteClick: (Note) -> Unit,
    private val onNoteLongClick: (Note) -> Unit
) : RecyclerView.Adapter<NotesAdapter.NoteViewHolder>() {

    private val notes = mutableListOf<Note>()
    private val selectedNotes = mutableSetOf<Note>()

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

            // Set up long click listener to enable selection mode
            binding.root.setOnLongClickListener {
                onNoteLongClick(note)
                true
            }

            // Show or hide selection circle based on selection state
            binding.selectionCircle.visibility = if (selectedNotes.contains(note)) View.VISIBLE else View.GONE
        }
    }

    fun toggleSelection(note: Note) {
        if (selectedNotes.contains(note)) {
            selectedNotes.remove(note)
        } else {
            selectedNotes.add(note)
        }
        notifyDataSetChanged()
    }

    fun getSelectedNotes(): Set<Note> = selectedNotes

    fun clearSelections() {
        selectedNotes.clear()
        notifyDataSetChanged()
    }
}
```

Step 2: Handle Long Press in MainActivity

In your MainActivity, handle the note long press event to toggle selection mode and show the delete button.
```
package com.example.blocodenotas

import android.content.Context
import android.content.Intent
import android.os.Bundle
import android.view.Menu
import android.view.MenuItem
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
    private var isSelectionMode = false

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
    }

    override fun onCreateOptionsMenu(menu: Menu?): Boolean {
        menuInflater.inflate(R.menu.main_menu, menu)
        return true
    }

    override fun onPrepareOptionsMenu(menu: Menu?): Boolean {
        menu?.findItem(R.id.action_delete)?.isVisible = isSelectionMode
        return super.onPrepareOptionsMenu(menu)
    }

    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        return when (item.itemId) {
            R.id.action_delete -> {
                deleteSelectedNotes()
                true
            }
            else -> super.onOptionsItemSelected(item)
        }
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
        isSelectionMode = true
        notesAdapter.toggleSelection(note)
        invalidateOptionsMenu()
    }

    private fun deleteSelectedNotes() {
        val selectedNotes = notesAdapter.getSelectedNotes()
        selectedNotes.forEach { notesAdapter.removeNote(it) }
        notesAdapter.clearSelections()
        isSelectionMode = false
        saveNotes()
        invalidateOptionsMenu()
    }
}
```

Step 3: Update Menu Resource

Create or update the menu resource file res/menu/main_menu.xml for the delete button:
```
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/action_delete"
        android:title="Delete"
        android:icon="@drawable/ic_delete"
        android:showAsAction="ifRoom"
        android:visible="false"/>
</menu>
```

Step 4: Update Note Item Layout
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="8dp">

    <!-- Add your existing note item views here -->

    <TextView
        android:id="@+id/textViewTitle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Title"
        android:textSize="18sp"
        android:textStyle="bold"/>

    <TextView
        android:id="@+id/textViewContent"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Content"
        android:textSize="16sp"/>

    <!-- Selection Circle -->
    <ImageView
        android:id="@+id/selectionCircle"
        android:layout_width="24dp"
        android:layout_height="24dp"
        android:layout_gravity="end"
        android:src="@drawable/ic_circle"
        android:visibility="gone"/>
</LinearLayout>
```

Update your note item layout (note_item.xml) to include a selection circle:
