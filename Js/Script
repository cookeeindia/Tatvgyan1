// =====================================================================
// Firebase ઇનિશિયલાઇઝેશન અને રેફરન્સ (index.html માં સેટ કરેલ છે)
// =====================================================================
const db = firebase.database(); // Firebase Realtime Database નો રેફરન્સ
const storage = firebase.storage(); // Firebase Storage નો રેફરન્સ
const auth = firebase.auth(); // Firebase Authentication નો રેફરન્સ

// =====================================================================
// DOM એલિમેન્ટ્સના રેફરન્સ (એપ શરૂ થાય ત્યારે આ બધા એલિમેન્ટ્સ HTML માં બનાવવામાં આવશે)
// =====================================================================
const appContainer = document.getElementById('app'); // મુખ્ય એપ કન્ટેનર
// addButton ને અહીંયા બનાવીએ છીએ, પણ તેને body માં DOMContentLoaded માં ઉમેરીશું
const addButton = document.createElement('div');
addButton.className = 'add-button';
addButton.innerHTML = '<span class="material-icons">add</span>';

let currentScreen = null; // કઈ સ્ક્રીન હાલમાં દેખાય છે તેનો ટ્રેક રાખવા માટે

// Home Screen Elements
let categoryCardsContainer;
let searchInput;
let searchButton;

// Listing Screen Elements
let notesList;
let listingTitle;
let backToHomeFromListingButton;

// Add/Edit Note Screen Elements
let noteForm;
let noteIdInput;
let subjectInput;
let bookInput;
let pageInput;
let paragraphTextarea;
let searchTagsInput;
let additionalFieldsDiv;
let yearInput;
let pushpaInput;
let categoryCheckboxesContainer;
let photoUploadContainer;
let photoInput;
let photoPreviewsContainer;
let cancelNoteFormButton;
let noteFormTitle;
let currentPhotos = []; // હાલમાં ફોર્મમાં અપલોડ કરેલા/પ્રિવ્યૂ કરેલા ફોટા

// Photo View Screen Elements
let photoGalleryContainer;
let backFromPhotoViewButton;
let editNoteFromPhotoViewButton;
let deleteNoteFromPhotoViewButton;
let photoViewTitle;

// Delete Confirmation Modal Elements
let deleteConfirmationModal;
let confirmDeleteButton;
let cancelDeleteButton;

// =====================================================================
// એપના ડેટા મોડેલ્સ
// =====================================================================

// નોંધ (Note) માટેનું ડેટા મોડેલ
class Note {
    constructor(id, subject, book, page, paragraph, searchTags, categories, photos, timestamp, year = null, pushpa = null) {
        this.id = id;
        this.subject = subject;
        this.book = book;
        this.page = page;
        this.paragraph = paragraph;
        this.searchTags = searchTags; // Array of strings
        this.categories = categories; // Array of category IDs
        this.photos = photos; // Array of photo objects {url: '...', order: 1}
        this.timestamp = timestamp; // ક્યારે બનાવ્યું/છેલ્લે અપડેટ કર્યું
        this.year = year; // "તત્વજ્ઞાન" પુસ્તક માટે
        this.pushpa = pushpa; // "તત્વજ્ઞાન" પુસ્તક માટે
    }
}

// કેટેગરી (Category) માટેનું ડેટા મોડેલ
class Category {
    constructor(id, name, iconName, isDefault = false) {
        this.id = id;
        this.name = name;
        this.iconName = iconName; // Material Icons માંથી આઇકોનનું નામ (દા.ત., 'menu_book')
        this.isDefault = isDefault; // શું આ યુઝર દ્વારા બનાવવામાં આવેલી ડિફોલ્ટ કેટેગરી છે?
    }
}

// =====================================================================
// ગ્લોબલ વેરીએબલ્સ
// =====================================================================
let allCategories = []; // બધી કેટેગરીઝનો સંગ્રહ
let allNotes = [];      // બધી નોંધોનો સંગ્રહ
let currentUserId = null; // Firebase Anonymous Auth યુઝર ID
let selectedNoteForEditDelete = null; // એડિટ કે ડિલીટ કરવા માટે પસંદ થયેલી નોંધ

// =====================================================================
// યુટિલિટી ફંક્શન્સ
// =====================================================================

// સ્ક્રીન બદલવા માટેનું ફંક્શન
function showScreen(screenId, data = {}) {
    // બધી સ્ક્રીનને છુપાવો
    document.querySelectorAll('.screen').forEach(screen => {
        screen.classList.remove('active');
    });

    // નવી સ્ક્રીનને બતાવો
    const targetScreen = document.getElementById(screenId);
    if (targetScreen) {
        targetScreen.classList.add('active');
        currentScreen = screenId; // વર્તમાન સ્ક્રીન અપડેટ કરો
    }

    // ડેટાના આધારે સ્ક્રીનનું કન્ટેન્ટ અપડેટ કરો
    if (screenId === 'addEditNoteScreen') {
        populateNoteForm(data.note);
    } else if (screenId === 'photoViewScreen') {
        displayPhotoView(data.note);
    } else if (screenId === 'listingScreen') {
        displayListingScreen(data.filter);
    } else if (screenId === 'homeScreen') {
        renderCategoryCards(); // ખાતરી કરો કે હોમ સ્ક્રીન પર કેટેગરીઝ રેન્ડર થાય
    }
}

// સ્ક્રીન કન્ટેનર બનાવવાનું અને રેફરન્સ મેળવવાનું ફંક્શન
function createScreenContainers() {
    // હોમ સ્ક્રીન
    const homeScreen = document.createElement('div');
    homeScreen.id = 'homeScreen';
    homeScreen.className = 'screen active'; // શરૂઆતમાં હોમ સ્ક્રીન સક્રિય રહેશે
    homeScreen.innerHTML = `
        <header>
            <h1>તત્વજ્ઞાન સંદર્ભ</h1>
        </header>
        <div class="search-container">
            <input type="text" id="searchInput" placeholder="વિષય અથવા ટેગ શોધો...">
            <button id="searchButton">શોધો</button>
        </div>
        <div class="category-cards" id="categoryCardsContainer">
            </div>
    `;
    appContainer.appendChild(homeScreen);

    // લિસ્ટિંગ સ્ક્રીન (નોંધોની યાદી બતાવવા માટે)
    const listingScreen = document.createElement('div');
    listingScreen.id = 'listingScreen';
    listingScreen.className = 'screen';
    listingScreen.innerHTML = `
        <header>
            <button id="backToHomeFromListing" class="material-icons">arrow_back</button>
            <h1 id="listingTitle">નોંધો</h1>
        </header>
        <ul id="notesList">
            </ul>
    `;
    appContainer.appendChild(listingScreen);

    // એડ/એડિટ નોંધ સ્ક્રીન
    const addEditNoteScreen = document.createElement('div');
    addEditNoteScreen.id = 'addEditNoteScreen';
    addEditNoteScreen.className = 'screen';
    addEditNoteScreen.innerHTML = `
        <header>
            <button id="backFromNoteForm" class="material-icons">arrow_back</button>
            <h1 id="noteFormTitle">નવી નોંધ ઉમેરો</h1>
        </header>
        <form id="noteForm">
            <input type="hidden" id="noteId"> <label for="subject">વિષય:</label>
            <input type="text" id="subject" required>

            <label for="book">પુસ્તક:</label>
            <input type="number" id="book" required>

            <label for="page">પૃષ્ઠ:</label>
            <input type="number" id="page" required>

            <label for="paragraph">પેરેગ્રાફ:</label>
            <textarea id="paragraph"></textarea>

            <label for="searchTags">સર્ચ ટેગ્સ (કોમા દ્વારા અલગ કરો):</label>
            <input type="text" id="searchTags">
            
            <div id="additionalFields" style="display: none;">
                <label for="year">વર્ષ:</label>
                <input type="number" id="year">

                <label for="pushpa">પુષ્પ:</label>
                <input type="number" id="pushpa">
            </div>

            <label for="categories">કેટેગરીઝ:</label>
            <div id="categoryCheckboxes">
                </div>

            <label for="photoUpload">ફોટા અપલોડ કરો (મહત્તમ 10):</label>
            <div class="photo-upload-container" id="photoUploadContainer">
                <input type="file" id="photoInput" accept="image/jpeg" multiple style="display: none;">
                <span class="material-icons" style="font-size: 3em;">cloud_upload</span><br>
                ફોટા અપલોડ કરવા ક્લિક કરો અથવા ખેંચો અને છોડો
            </div>
            <div class="photo-previews" id="photoPreviewsContainer">
                </div>

            <div class="form-actions">
                <button type="submit" class="save-button">સાચવો</button>
                <button type="button" class="cancel-button" id="cancelNoteForm">રદ કરો</button>
            </div>
        </form>
    `;
    appContainer.appendChild(addEditNoteScreen);

    // ફોટો વ્યૂ સ્ક્રીન
    const photoViewScreen = document.createElement('div');
    photoViewScreen.id = 'photoViewScreen';
    photoViewScreen.className = 'screen';
    photoViewScreen.innerHTML = `
        <header>
            <button id="backFromPhotoView" class="material-icons">arrow_back</button>
            <h1 id="photoViewTitle">ફોટા</h1>
            <button id="editNoteFromPhotoView" class="material-icons">edit</button>
            <button id="deleteNoteFromPhotoView" class="material-icons">delete</button>
        </header>
        <div class="photo-gallery" id="photoGalleryContainer">
            </div>
    `;
    appContainer.appendChild(photoViewScreen);

    // ડિલીટ કન્ફર્મેશન મોડલ (શરૂઆતમાં છુપાયેલું)
    const deleteConfirmationModalElement = document.createElement('div'); // Rename to avoid conflict
    deleteConfirmationModalElement.id = 'deleteConfirmationModal';
    deleteConfirmationModalElement.className = 'modal-overlay';
    deleteConfirmationModalElement.innerHTML = `
        <div class="modal-content">
            <h3>નોંધ કાઢી નાખો</h3>
            <p>શું તમે ખરેખર આ નોંધ કાઢી નાખવા માંગો છો? આ ક્રિયા પૂર્વવત કરી શકાશે નહીં.</p>
            <div class="modal-buttons">
                <button class="confirm-button" id="confirmDeleteButton">હા, કાઢી નાખો</button>
                <button class="cancel-button" id="cancelDeleteButton">રદ કરો</button>
            </div>
        </div>
    `;
    document.body.appendChild(deleteConfirmationModalElement); // body માં ઉમેરો

    // DOM એલિમેન્ટ્સના રેફરન્સ મેળવો (createScreenContainers પછી)
    categoryCardsContainer = document.getElementById('categoryCardsContainer');
    searchInput = document.getElementById('searchInput');
    searchButton = document.getElementById('searchButton');
    notesList = document.getElementById('notesList');
    listingTitle = document.getElementById('listingTitle');
    backToHomeFromListingButton = document.getElementById('backToHomeFromListing');
    noteForm = document.getElementById('noteForm');
    noteIdInput = document.getElementById('noteId');
    subjectInput = document.getElementById('subject');
    bookInput = document.getElementById('book');
    pageInput = document.getElementById('page');
    paragraphTextarea = document.getElementById('paragraph');
    searchTagsInput = document.getElementById('searchTags');
    additionalFieldsDiv = document.getElementById('additionalFields');
    yearInput = document.getElementById('year');
    pushpaInput = document.getElementById('pushpa');
    categoryCheckboxesContainer = document.getElementById('categoryCheckboxes');
    photoUploadContainer = document.getElementById('photoUploadContainer');
    photoInput = document.getElementById('photoInput');
    photoPreviewsContainer = document.getElementById('photoPreviewsContainer');
    cancelNoteFormButton = document.getElementById('cancelNoteForm');
    noteFormTitle = document.getElementById('noteFormTitle');
    photoGalleryContainer = document.getElementById('photoGalleryContainer');
    backFromPhotoViewButton = document.getElementById('backFromPhotoView');
    editNoteFromPhotoViewButton = document.getElementById('editNoteFromPhotoView');
    deleteNoteFromPhotoViewButton = document.getElementById('deleteNoteFromPhotoView');
    photoViewTitle = document.getElementById('photoViewTitle');
    deleteConfirmationModal = document.getElementById('deleteConfirmationModal');
    confirmDeleteButton = document.getElementById('confirmDeleteButton');
    cancelDeleteButton = document.getElementById('cancelDeleteButton');
}

// =====================================================================
// હોમ સ્ક્રીન લોજિક
// =====================================================================

// કેટેગરી કાર્ડ્સને રેન્ડર કરવા માટેનું ફંક્શન
function renderCategoryCards() {
    categoryCardsContainer.innerHTML = ''; // હાલના કાર્ડ્સ સાફ કરો
    
    // પૂર્વનિર્ધારિત કેટેગરીઝ
    const defaultCategories = [
        new Category('tatvgyan', 'તત્વજ્ઞાન', 'self_improvement', true),
        new Category('sahitya', 'સાહિત્ય', 'book', true),
        new Category('itihas', 'ઇતિહાસ', 'museum', true),
        new Category('vigyan', 'વિજ્ઞાન', 'science', true),
        new Category('kala', 'કલા', 'palette', true)
    ];

    // જો Firebase માંથી કોઈ કેટેગરી લોડ ન થાય તો ડિફોલ્ટ કેટેગરીઝ ઉમેરો
    if (allCategories.length === 0) {
        allCategories = defaultCategories;
        // વૈકલ્પિક: તમે આ ડિફોલ્ટ કેટેગરીઝને Firebase માં પણ ઉમેરી શકો છો
        // defaultCategories.forEach(cat => db.ref('categories/' + cat.id).set(cat));
    }

    allCategories.forEach(category => {
        const card = document.createElement('div');
        card.className = 'category-card';
        card.dataset.categoryId = category.id; // ક્લિક પર ઉપયોગ કરવા માટે ID સ્ટોર કરો
        card.innerHTML = `
            <span class="material-icons">${category.iconName || 'category'}</span>
            <h3>${category.name}</h3>
        `;
        categoryCardsContainer.appendChild(card);

        // કેટેગરી કાર્ડ પર ક્લિક લિસનર ઉમેરો
        card.addEventListener('click', () => {
            showScreen('listingScreen', { filter: { categoryId: category.id, categoryName: category.name } });
        });
    });
}

// સર્ચ બટન પર ક્લિક લિસનર
// આ લિસનર DOMContentLoaded માં ઉમેરવામાં આવશે
// searchButton.addEventListener('click', ...);

// =====================================================================
// લિસ્ટિંગ સ્ક્રીન લોજિક
// =====================================================================

// લિસ્ટિંગ સ્ક્રીન બતાવવા અને નોંધોને ફિલ્ટર કરવા માટેનું ફંક્શન
function displayListingScreen(filter = {}) {
    notesList.innerHTML = ''; // હાલની યાદી સાફ કરો
    
    let filteredNotes = [...allNotes]; // બધી નોંધોની એક કોપી લો

    // ફિલ્ટર લોજિક
    if (filter.categoryId) {
        filteredNotes = filteredNotes.filter(note => note.categories && note.categories.includes(filter.categoryId));
        listingTitle.textContent = allCategories.find(cat => cat.id === filter.categoryId)?.name || 'કેટેગરી નોંધો';
        // જો "તત્વજ્ઞાન" કેટેગરી હોય, તો વર્ષ/પુષ્પના આધારે સોર્ટ કરો
        if (filter.categoryId === 'tatvgyan') {
            filteredNotes.sort((a, b) => {
                if (a.year !== b.year) return (a.year || 0) - (b.year || 0); // વર્ષ દ્વારા સોર્ટ
                return (a.pushpa || 0) - (b.pushpa || 0); // પછી પુષ્પ દ્વારા સોર્ટ
            });
        }
    } else if (filter.searchTerm) {
        const searchTermLower = filter.searchTerm.toLowerCase();
        filteredNotes = filteredNotes.filter(note => 
            (note.subject && note.subject.toLowerCase().includes(searchTermLower)) ||
            (note.paragraph && note.paragraph.toLowerCase().includes(searchTermLower)) ||
            (note.searchTags && note.searchTags.some(tag => tag.toLowerCase().includes(searchTermLower))) ||
            (note.book && String(note.book).toLowerCase().includes(searchTermLower)) || // પુસ્તક નંબર પણ સર્ચ કરી શકાય
            (note.year && String(note.year).toLowerCase().includes(searchTermLower)) ||
            (note.pushpa && String(note.pushpa).toLowerCase().includes(searchTermLower))
        );
        listingTitle.textContent = `"${filter.searchTerm}" માટેના પરિણામો`;
    } else {
        // કોઈ ફિલ્ટર ન હોય તો બધી નોંધો બતાવો (તાજેતરની પહેલાથી સોર્ટ કરેલી)
        listingTitle.textContent = 'બધી નોંધો';
    }

    if (filteredNotes.length === 0) {
        const noNotesItem = document.createElement('li');
        noNotesItem.textContent = 'કોઈ નોંધો મળી નથી.';
        notesList.appendChild(noNotesItem);
    } else {
        filteredNotes.forEach(note => {
            const listItem = document.createElement('li');
            listItem.dataset.noteId = note.id;
            
            const categoriesNames = note.categories ? note.categories.map(catId => 
                allCategories.find(cat => cat.id === catId)?.name || catId // જો નામ ન મળે તો ID બતાવો
            ).join(', ') : '';

            listItem.innerHTML = `
                <div class="note-details">
                    <h4>${note.subject || 'અજ્ઞાત વિષય'}</h4>
                    <p>પુસ્તક: ${note.book || '-'}, પૃષ્ઠ: ${note.page || '-'} ${note.year ? `(વર્ષ: ${note.year}, પુષ્પ: ${note.pushpa})` : ''}</p>
                    <p>કેટેગરી: ${categoriesNames}</p>
                    <p>${note.paragraph ? note.paragraph.substring(0, 100) + '...' : 'કોઈ પેરેગ્રાફ નથી.'}</p>
                </div>
                <div class="note-actions">
                    <span class="material-icons edit-note" data-note-id="${note.id}">edit</span>
                    <span class="material-icons delete-note" data-note-id="${note.id}">delete</span>
                </div>
            `;
            notesList.appendChild(listItem);

            // લિસ્ટ આઇટમ પર ક્લિક લિસનર (ફોટો વ્યૂ અથવા વિગત)
            listItem.querySelector('.note-details').addEventListener('click', () => {
                // જો નોંધમાં ફોટા હોય તો ફોટો વ્યૂ સ્ક્રીન બતાવો, નહીંતર એડિટ ફોર્મ
                if (note.photos && note.photos.length > 0) {
                    selectedNoteForEditDelete = note; // પસંદ કરેલી નોટ સેટ કરો
                    showScreen('photoViewScreen', { note: note });
                } else {
                    selectedNoteForEditDelete = note; // પસંદ કરેલી નોટ સેટ કરો
                    showScreen('addEditNoteScreen', { note: note });
                }
            });

            // એડિટ બટન પર ક્લિક લિસનર
            listItem.querySelector('.edit-note').addEventListener('click', (event) => {
                event.stopPropagation(); // લિસ્ટ આઇટમ ક્લિકને રોકો
                selectedNoteForEditDelete = note; // પસંદ કરેલી નોટ સેટ કરો
                showScreen('addEditNoteScreen', { note: note });
            });

            // ડિલીટ બટન પર ક્લિક લિસનર
            listItem.querySelector('.delete-note').addEventListener('click', (event) => {
                event.stopPropagation(); // લિસ્ટ આઇટમ ક્લિકને રોકો
                selectedNoteForEditDelete = note; // પસંદ કરેલી નોટ સેટ કરો
                showDeleteConfirmation(note); // ડિલીટ કન્ફર્મેશન મોડલ બતાવો
            });
        });
    }
}

// લિસ્ટિંગ સ્ક્રીનમાંથી હોમ સ્ક્રીન પર પાછા જવા માટેનું બટન
// આ લિસનર DOMContentLoaded માં ઉમેરવામાં આવશે
// backToHomeFromListingButton.addEventListener('click', ...);

// =====================================================================
// એડ/એડિટ નોંધ સ્ક્રીન લોજિક (Form Handling)
// =====================================================================

// કેટેગરી ચેકબોક્સ રેન્ડર કરો
function renderCategoryCheckboxes(selectedCategories = []) {
    categoryCheckboxesContainer.innerHTML = '';
    allCategories.forEach(category => {
        const checkboxDiv = document.createElement('div');
        checkboxDiv.innerHTML = `
            <input type="checkbox" id="cat-${category.id}" value="${category.id}" ${selectedCategories.includes(category.id) ? 'checked' : ''}>
            <label for="cat-${category.id}">${category.name}</label>
        `;
        categoryCheckboxesContainer.appendChild(checkboxDiv);
    });

    // "તત્વજ્ઞાન" કેટેગરી માટે વધારાના ફિલ્ડ્સ ટોગલ કરો
    const tatvgyanCheckbox = document.querySelector('#categoryCheckboxes input[value="tatvgyan"]');
    if (tatvgyanCheckbox) {
        // ખાતરી કરો કે લિસનર માત્ર એક જ વાર ઉમેરાય
        tatvgyanCheckbox.removeEventListener('change', toggleAdditionalFields); 
        tatvgyanCheckbox.addEventListener('change', toggleAdditionalFields);
    }
    toggleAdditionalFields(); // પ્રારંભિક અવસ્થા સેટ કરો
}

// "તત્વજ્ઞાન" કેટેગરી પસંદ હોય તો વર્ષ/પુષ્પ ફિલ્ડ્સ બતાવો/છુપાવો
function toggleAdditionalFields() {
    const tatvgyanCheckbox = document.querySelector('#categoryCheckboxes input[value="tatvgyan"]');
    if (tatvgyanCheckbox && tatvgyanCheckbox.checked) {
        additionalFieldsDiv.style.display = 'block';
    } else {
        additionalFieldsDiv.style.display = 'none';
        yearInput.value = '';
        pushpaInput.value = '';
    }
}

// નોંધ ફોર્મને પોપ્યુલેટ કરો (એડિટ કરવા માટે)
function populateNoteForm(note) {
    noteFormTitle.textContent = 'નોંધ એડિટ કરો';
    noteIdInput.value = note.id || '';
    subjectInput.value = note.subject || '';
    bookInput.value = note.book || '';
    pageInput.value = note.page || '';
    paragraphTextarea.value = note.paragraph || '';
    searchTagsInput.value = (note.searchTags || []).join(', ');
    yearInput.value = note.year || '';
    pushpaInput.value = note.pushpa || '';

    renderCategoryCheckboxes(note.categories || []); // કેટેગરી ચેકબોક્સ ભરો
    currentPhotos = note.photos || []; // હાલના ફોટા સેટ કરો
    renderPhotoPreviews(); // ફોટો પ્રિવ્યૂ બતાવો
    toggleAdditionalFields(); // "તત્વજ્ઞાન" ફિલ્ડ્સની સ્થિતિ અપડેટ કરો
}

// ફોટો પ્રિવ્યૂ રેન્ડર કરો
function renderPhotoPreviews() {
    photoPreviewsContainer.innerHTML = '';
    currentPhotos.sort((a, b) => a.order - b.order); // ઓર્ડર દ્વારા સોર્ટ કરો

    currentPhotos.forEach((photo, index) => {
        const photoPreviewItem = document.createElement('div');
        photoPreviewItem.className = 'photo-preview-item';
        photoPreviewItem.draggable = true; // ડ્રેગ અને ડ્રોપ સક્ષમ કરો
        photoPreviewItem.dataset.index = index; // ઓર્ડર માટે ઇન્ડેક્સ સ્ટોર કરો

        photoPreviewItem.innerHTML = `
            <img src="${photo.url}" alt="નોંધ ફોટો">
            <span class="material-icons remove-photo">close</span>
        `;
        photoPreviewsContainer.appendChild(photoPreviewItem);

        // ફોટો દૂર કરવાનું બટન
        photoPreviewItem.querySelector('.remove-photo').addEventListener('click', () => {
            removePhoto(index);
        });

        // ડ્રેગ અને ડ્રોપ ઇવેન્ટ્સ
        photoPreviewItem.addEventListener('dragstart', handleDragStart);
        photoPreviewItem.addEventListener('dragover', handleDragOver);
        photoPreviewItem.addEventListener('drop', handleDrop);
        photoPreviewItem.addEventListener('dragend', handleDragEnd);
    });
}

// ફોટો દૂર કરો
function removePhoto(indexToRemove) {
    const photoToRemove = currentPhotos[indexToRemove];
    if (photoToRemove && photoToRemove.url.startsWith('gs://')) { // જો Firebase Storage URL હોય
        const storageRef = storage.refFromURL(photoToRemove.url);
        storageRef.delete().then(() => {
            console.log('Photo deleted from Storage:', photoToRemove.url);
            currentPhotos.splice(indexToRemove, 1);
            // બાકીના ફોટાનો ઓર્ડર ફરીથી સેટ કરો
            currentPhotos.forEach((photo, i) => photo.order = i);
            renderPhotoPreviews();
            alert('ફોટો સફળતાપૂર્વક કાઢી નાખ્યો.');
        }).catch((error) => {
            console.error('Error deleting photo from Storage:', error);
            alert('ફોટો કાઢી નાખવામાં ભૂલ આવી.');
        });
    } else { // જો નવો અપલોડ થયેલો ફોટો હોય જે હજુ સ્ટોરેજમાં નથી
        currentPhotos.splice(indexToRemove, 1);
        currentPhotos.forEach((photo, i) => photo.order = i);
        renderPhotoPreviews();
        alert('ફોટો સફળતાપૂર્વક દૂર કર્યો.');
    }
}

// ફોટો અપલોડ હેન્ડલર
async function handlePhotoUpload(event) {
    const files = event.target.files;
    if (currentPhotos.length + files.length > 10) {
        alert('તમે મહત્તમ 10 ફોટા અપલોડ કરી શકો છો.');
        return;
    }

    for (const file of files) {
        if (!file.type.startsWith('image/jpeg')) {
            alert('માત્ર JPG/JPEG ફાઇલોને મંજૂરી છે.');
            continue;
        }
        if (file.size > 5 * 1024 * 1024) { // 5MB મર્યાદા
            alert(`"${file.name}" 5MB થી મોટી છે. કૃપા કરીને નાની ફાઇલ પસંદ કરો.`);
            continue;
        }

        const reader = new FileReader();
        reader.onload = (e) => {
            currentPhotos.push({ url: e.target.result, file: file, order: currentPhotos.length });
            renderPhotoPreviews();
        };
        reader.readAsDataURL(file);
    }
    photoInput.value = ''; // ફાઇલ ઇનપુટ સાફ કરો
}

// ડ્રેગ એન્ડ ડ્રોપ હેન્ડલર્સ
let draggedItem = null;

function handleDragStart(e) {
    draggedItem = this;
    e.dataTransfer.effectAllowed = 'move';
    e.dataTransfer.setData('text/html', this.innerHTML);
    e.dataTransfer.setData('text/plain', this.dataset.index); // ડ્રેગ કરેલા આઇટમનો ઇન્ડેક્સ
    this.classList.add('dragging');
}

function handleDragOver(e) {
    e.preventDefault(); // ડ્રોપને મંજૂરી આપવા માટે જરૂરી
    e.dataTransfer.dropEffect = 'move';
    const target = e.target.closest('.photo-preview-item');
    if (target && target !== draggedItem) {
        const rect = target.getBoundingClientRect();
        const middleX = rect.left + rect.width / 2;
        if (e.clientX > middleX) {
            target.style.borderRight = '2px solid #007bff';
            target.style.borderLeft = '';
        } else {
            target.style.borderLeft = '2px solid #007bff';
            target.style.borderRight = '';
        }
    }
}

function handleDrop(e) {
    e.preventDefault();
    if (draggedItem && draggedItem !== this) {
        const dropIndex = parseInt(this.dataset.index);
        const dragIndex = parseInt(draggedItem.dataset.index);

        // પ્રિવ્યૂ બોર્ડર સાફ કરો
        this.style.borderLeft = '';
        this.style.borderRight = '';

        // currentPhotos એરેમાં ઓર્ડર બદલો
        const [removed] = currentPhotos.splice(dragIndex, 1);
        currentPhotos.splice(dropIndex, 0, removed);

        // નવા ઓર્ડર સેટ કરો
        currentPhotos.forEach((photo, i) => photo.order = i);
        renderPhotoPreviews(); // પ્રિવ્યૂ ફરીથી રેન્ડર કરો
    }
}

function handleDragEnd() {
    this.classList.remove('dragging');
    draggedItem = null;
    document.querySelectorAll('.photo-preview-item').forEach(item => {
        item.style.borderLeft = '';
        item.style.borderRight = '';
    });
}


// નોંધ સેવ/અપડેટ કરો
async function saveNote(event) {
    event.preventDefault();

    const noteId = noteIdInput.value || db.ref('users/' + currentUserId + '/notes').push().key; // નવો ID અથવા હાલનો ID
    const subject = subjectInput.value.trim();
    const book = parseInt(bookInput.value);
    const page = parseInt(pageInput.value);
    const paragraph = paragraphTextarea.value.trim();
    const searchTags = searchTagsInput.value.split(',').map(tag => tag.trim()).filter(tag => tag !== '');
    const selectedCategories = Array.from(categoryCheckboxesContainer.querySelectorAll('input:checked')).map(checkbox => checkbox.value);
    const year = yearInput.value ? parseInt(yearInput.value) : null;
    const pushpa = pushpaInput.value ? parseInt(pushpaInput.value) : null;
    const timestamp = Date.now();

    if (!subject || !book || !page || selectedCategories.length === 0) {
        alert('કૃપા કરીને વિષય, પુસ્તક, પૃષ્ઠ અને ઓછામાં ઓછી એક કેટેગરી દાખલ કરો.');
        return;
    }

    // ફોટા અપલોડ કરો અને URLs મેળવો
    const uploadedPhotoUrls = [];
    for (let i = 0; i < currentPhotos.length; i++) {
        const photo = currentPhotos[i];
        if (photo.file) { // જો આ નવી અપલોડ થયેલી ફાઇલ હોય
            const photoRef = storage.ref('users/' + currentUserId + '/notes/' + noteId + '/' + Date.now() + '_' + photo.file.name);
            const snapshot = await photoRef.put(photo.file);
            const downloadURL = await snapshot.ref.getDownloadURL();
            uploadedPhotoUrls.push({ url: downloadURL, order: photo.order });
        } else { // જો આ પહેલેથી અપલોડ થયેલી ફાઇલ હોય
            uploadedPhotoUrls.push(photo);
        }
    }

    const newNote = new Note(
        noteId,
        subject,
        book,
        page,
        paragraph,
        searchTags,
        selectedCategories,
        uploadedPhotoUrls,
        timestamp,
        year,
        pushpa
    );

    try {
        await db.ref('users/' + currentUserId + '/notes/' + noteId).set(newNote);
        alert('નોંધ સફળતાપૂર્વક સાચવી!');
        await loadNotes(); // નોંધોને ફરીથી લોડ કરો
        showScreen('listingScreen'); // યાદી સ્ક્રીન પર પાછા ફરો
    } catch (error) {
        console.error("Error saving note:", error);
        alert('નોંધ સાચવતી વખતે ભૂલ આવી: ' + error.message);
    }
}

// નોંધ ફોર્મ રદ કરો
function cancelNoteForm() {
    showScreen('homeScreen'); // હોમ સ્ક્રીન પર પાછા ફરો
    noteForm.reset();
    currentPhotos = []; // ફોટા સાફ કરો
    photoPreviewsContainer.innerHTML = '';
    additionalFieldsDiv.style.display = 'none'; // વધારાના ફિલ્ડ્સ છુપાવો
}

// =====================================================================
// ફોટો વ્યૂ સ્ક્રીન લોજિક
// =====================================================================

// ફોટો વ્યૂ સ્ક્રીન પર ફોટા પ્રદર્શિત કરો
function displayPhotoView(note) {
    photoViewTitle.textContent = `${note.subject || 'નોંધ'} ના ફોટા`;
    photoGalleryContainer.innerHTML = ''; // હાલની ગેલેરી સાફ કરો

    if (!note.photos || note.photos.length === 0) {
        photoGalleryContainer.innerHTML = '<p>આ નોંધ માટે કોઈ ફોટા નથી.</p>';
        return;
    }

    note.photos.sort((a, b) => a.order - b.order); // ઓર્ડર દ્વારા સોર્ટ કરો

    note.photos.forEach(photo => {
        const photoItem = document.createElement('div');
        photoItem.className = 'photo-item';
        const img = document.createElement('img');
        img.src = photo.url;
        img.alt = 'નોંધ ફોટો';
        img.className = 'zoomable-image'; // ઝૂમ માટે ક્લાસ
        photoItem.appendChild(img);
        photoGalleryContainer.appendChild(photoItem);

        // ઝૂમ કાર્યક્ષમતા
        let isZoomed = false;
        img.addEventListener('click', (e) => {
            if (!isZoomed) {
                img.style.transform = 'scale(2)'; // 2x ઝૂમ
                img.style.cursor = 'zoom-out';
            } else {
                img.style.transform = 'scale(1)'; // નોર્મલ
                img.style.cursor = 'zoom-in';
            }
            isZoomed = !isZoomed;
        });
    });
}

// ફોટો વ્યૂ સ્ક્રીનમાંથી પાછા જાઓ
// આ લિસનર DOMContentLoaded માં ઉમેરવામાં આવશે
// backFromPhotoViewButton.addEventListener('click', ...);

// ફોટો વ્યૂમાંથી નોંધ એડિટ કરો
// આ લિસનર DOMContentLoaded માં ઉમેરવામાં આવશે
// editNoteFromPhotoViewButton.addEventListener('click', ...);

// ફોટો વ્યૂમાંથી નોંધ ડિલીટ કરો
// આ લિસનર DOMContentLoaded માં ઉમેરવામાં આવશે
// deleteNoteFromPhotoViewButton.addEventListener('click', ...);


// =====================================================================
// ડિલીટ કન્ફર્મેશન મોડલ લોજિક
// =====================================================================

function showDeleteConfirmation(note) {
    selectedNoteForEditDelete = note; // ખાતરી કરો કે કઈ નોંધ ડિલીટ કરવાની છે
    deleteConfirmationModal.style.display = 'flex'; // મોડલ બતાવો
}

function hideDeleteConfirmation() {
    deleteConfirmationModal.style.display = 'none'; // મોડલ છુપાવો
    selectedNoteForEditDelete = null; // પસંદગી સાફ કરો
}

async function deleteNoteConfirmed() {
    if (!selectedNoteForEditDelete || !currentUserId) {
        console.error("No note selected for deletion or user not authenticated.");
        hideDeleteConfirmation();
        return;
    }

    const noteToDelete = selectedNoteForEditDelete;
    try {
        // Firebase Storage માંથી ફોટા ડિલીટ કરો
        if (noteToDelete.photos && noteToDelete.photos.length > 0) {
            for (const photo of noteToDelete.photos) {
                if (photo.url.startsWith('gs://') || photo.url.startsWith('https://firebasestorage.googleapis.com/')) {
                    try {
                        const photoRef = storage.refFromURL(photo.url);
                        await photoRef.delete();
                        console.log('Deleted photo from storage:', photo.url);
                    } catch (storageError) {
                        // જો ફોટો ન મળે અથવા અન્ય ભૂલ હોય તો પણ ચાલુ રાખો
                        console.warn('Could not delete photo from storage (might not exist):', photo.url, storageError);
                    }
                }
            }
        }

        // Firebase Realtime Database માંથી નોંધ ડિલીટ કરો
        await db.ref('users/' + currentUserId + '/notes/' + noteToDelete.id).remove();
        alert('નોંધ સફળતાપૂર્વક કાઢી નાખી!');
        hideDeleteConfirmation();
        await loadNotes(); // નોંધોને ફરીથી લોડ કરો
        showScreen('homeScreen'); // હોમ સ્ક્રીન પર પાછા ફરો
    } catch (error) {
        console.error("Error deleting note:", error);
        alert('નોંધ કાઢી નાખવામાં ભૂલ આવી: ' + error.message);
        hideDeleteConfirmation();
    }
}

// =====================================================================
// Firebase સંબંધિત ફંક્શન્સ
// =====================================================================

// Firebase માંથી કેટેગરીઝ લોડ કરો
async function loadCategories() {
    try {
        const snapshot = await db.ref('categories').once('value');
        const categoriesData = snapshot.val();
        allCategories = [];
        if (categoriesData) {
            for (const id in categoriesData) {
                allCategories.push(new Category(id, categoriesData[id].name, categoriesData[id].iconName, categoriesData[id].isDefault));
            }
        }
        console.log('Categories loaded:', allCategories);
        // કેટેગરીઝ લોડ થયા પછી હોમ સ્ક્રીન રેન્ડર કરો (જો હોમ સ્ક્રીન સક્રિય હોય)
        if (currentScreen === 'homeScreen' || currentScreen === null) { // શરૂઆતમાં currentScreen null હોઈ શકે છે
            renderCategoryCards();
        }
    } catch (error) {
        console.error("Error loading categories:", error);
    }
}

// Firebase માંથી નોંધો લોડ કરો (વર્તમાન યુઝર માટે)
async function loadNotes() {
    if (!currentUserId) {
        console.warn("User not authenticated yet, cannot load notes.");
        return;
    }
    try {
        // યુઝરના UID હેઠળની નોંધો લોડ કરો
        const snapshot = await db.ref('users/' + currentUserId + '/notes').once('value');
        const notesData = snapshot.val();
        allNotes = [];
        if (notesData) {
            for (const id in notesData) {
                const note = notesData[id];
                allNotes.push(new Note(id, note.subject, note.book, note.page, note.paragraph, note.searchTags || [], note.categories || [], note.photos || [], note.timestamp, note.year, note.pushpa));
            }
        }
        allNotes.sort((a, b) => b.timestamp - a.timestamp); // તાજેતરની નોંધો પહેલા
        console.log('Notes loaded:', allNotes);
        // જો લિસ્ટિંગ સ્ક્રીન સક્રિય હોય, તો તેને અપડેટ કરો
        if (currentScreen === 'listingScreen') {
            displayListingScreen(selectedNoteForEditDelete ? { categoryId: selectedNoteForEditDelete.categories[0] } : {}); // જો કોઈ નોટ પસંદ હોય તો તેની કેટેગરી ફિલ્ટર કરો
        }
    } catch (error) {
        console.error("Error loading notes:", error);
    }
}

// =====================================================================
// Firebase Anonymous Authentication
// =====================================================================

// યુઝરને અનામી રીતે સાઇન ઇન કરો
async function signInAnonymously() {
    try {
        const userCredential = await auth.signInAnonymously();
        currentUserId = userCredential.user.uid;
        console.log('Signed in anonymously with UID:', currentUserId);
        // યુઝર ID મળ્યા પછી ડેટા લોડ કરો
        await loadCategories(); // કેટેગરીઝ લોડ કરો
        await loadNotes();      // નોંધો લોડ કરો
        // એપ શરૂ કરો (હોમ સ્ક્રીન બતાવો)
        showScreen('homeScreen');
    } catch (error) {
        console.error("Error during anonymous sign-in:", error);
        alert("એપ શરૂ કરવામાં ભૂલ આવી. કૃપા કરીને ફરી પ્રયાસ કરો.");
    }
}

// =====================================================================
// એપ શરૂ કરવાની લોજિક (DOMContentLoaded)
// =====================================================================

document.addEventListener('DOMContentLoaded', () => {
    console.log('DOM Content Loaded. Initializing app...');
    // વિવિધ સ્ક્રીનના કન્ટેનર HTML માં ઉમેરો અને તેમના DOM રેફરન્સ મેળવો
    createScreenContainers();
    
    // એડ બટનને body માં અહીં ઉમેરો, જેથી DOM fully loaded હોય
    document.body.appendChild(addButton); 

    // ================= ઇવેન્ટ લિસનર્સ ===================

    // હોમ સ્ક્રીન સર્ચ બટન
    searchButton.addEventListener('click', () => {
        const searchTerm = searchInput.value.trim();
        if (searchTerm) {
            showScreen('listingScreen', { filter: { searchTerm: searchTerm } });
        } else {
            alert('કૃપા કરીને શોધવા માટે કંઈક દાખલ કરો.');
        }
    });

    // લિસ્ટિંગ સ્ક્રીન બેક બટન
    backToHomeFromListingButton.addEventListener('click', () => {
        showScreen('homeScreen');
    });

    // એડ/એડિટ બટન (ફ્લોટિંગ બટન)
    addButton.addEventListener('click', () => {
        showScreen('addEditNoteScreen'); // નવી નોંધ ઉમેરવા માટે ફોર્મ બતાવો
        noteFormTitle.textContent = 'નવી નોંધ ઉમેરો';
        noteIdInput.value = ''; // ID સાફ કરો
        noteForm.reset(); // ફોર્મ રીસેટ કરો
        currentPhotos = []; // ફોટા સાફ કરો
        photoPreviewsContainer.innerHTML = ''; // ફોટો પ્રિવ્યૂ સાફ કરો
        additionalFieldsDiv.style.display = 'none'; // તત્વજ્ઞાન ફિલ્ડ્સ છુપાવો
        
        // કેટેગરી ચેકબોક્સને અનચેક કરો અને ફરીથી રેન્ડર કરો
        renderCategoryCheckboxes([]);
        // "તત્વજ્ઞાન" કેટેગરીના ચેકબોક્સ પર લિસનર ફરીથી સેટ કરો (renderCategoryCheckboxes માં થાય છે)
    });

    // ફોટો અપલોડ કન્ટેનર ક્લિક
    photoUploadContainer.addEventListener('click', () => {
        photoInput.click(); // છુપાયેલા ફાઇલ ઇનપુટને ક્લિક કરો
    });

    // ફોટો ઇનપુટ ચેન્જ
    photoInput.addEventListener('change', handlePhotoUpload);

    // ડ્રેગ અને ડ્રોપ માટે body પર લિસનર્સ (ફાઇલ ડ્રોપ કરવા માટે)
    photoUploadContainer.addEventListener('dragover', (e) => {
        e.preventDefault();
        photoUploadContainer.style.backgroundColor = '#e0e0e0';
    });

    photoUploadContainer.addEventListener('dragleave', () => {
        photoUploadContainer.style.backgroundColor = '';
    });

    photoUploadContainer.addEventListener('drop', (e) => {
        e.preventDefault();
        photoUploadContainer.style.backgroundColor = '';
        const files = e.dataTransfer.files;
        handlePhotoUpload({ target: { files: files } });
    });


    // નોંધ ફોર્મ સબમિટ
    noteForm.addEventListener('submit', saveNote);

    // નોંધ ફોર્મ રદ કરો બટન
    cancelNoteFormButton.addEventListener('click', cancelNoteForm);

    // ફોટો વ્યૂ સ્ક્રીનમાંથી પાછા જાઓ
    backFromPhotoViewButton.addEventListener('click', () => {
        showScreen('listingScreen', { filter: { categoryId: selectedNoteForEditDelete?.categories[0] || '' } }); // પાછળ ફિલ્ટર સાથે જાઓ
    });

    // ફોટો વ્યૂમાંથી નોંધ એડિટ કરો
    editNoteFromPhotoViewButton.addEventListener('click', () => {
        if (selectedNoteForEditDelete) {
            showScreen('addEditNoteScreen', { note: selectedNoteForEditDelete });
        }
    });

    // ફોટો વ્યૂમાંથી નોંધ ડિલીટ કરો
    deleteNoteFromPhotoViewButton.addEventListener('click', () => {
        if (selectedNoteForEditDelete) {
            showDeleteConfirmation(selectedNoteForEditDelete);
        }
    });

    // ડિલીટ કન્ફર્મેશન મોડલ બટન્સ
    confirmDeleteButton.addEventListener('click', deleteNoteConfirmed);
    cancelDeleteButton.addEventListener('click', hideDeleteConfirmation);


    // અનામી રીતે સાઇન ઇન કરો અને પછી ડેટા લોડ કરો
    signInAnonymously();
});
