<script>
  import { Plus, Trash2, ImagePlus, Download } from 'lucide-svelte';
  import { onMount, tick } from 'svelte';
  import { fade } from 'svelte/transition';

  let appState = {
    company: { name: '', logo: '' },
    invoice: {
      number: '',
      created: '',
      due: '',
      from: '',
      fromDetailsVisible: false,
      to: '',
      toDetailsVisible: false
    },
    items: [],
    taxPercent: '',
    discountPercent: '',
    currency: 'USD'
  };

  let currencyOptions = [
    { code: 'USD', symbol: '$', name: 'US Dollar', flag: '🇺🇸' },
    { code: 'EUR', symbol: '€', name: 'Euro', flag: '🇪🇺' },
    // Add more currencies as needed
  ];

  function getCurrencySymbol(code) {
    const currency = currencyOptions.find((c) => c.code === code);
    return currency ? currency.symbol : '';
  }

  function save() {
    localStorage.setItem('invoiceData', JSON.stringify(appState));
  }

  function addItem() {
    if (itemDesc.trim() !== '' && itemPrice > 0 && itemQty > 0) {
      appState.items = [
        ...appState.items,
        {
          desc: itemDesc.trim(),
          price: parseFloat(itemPrice).toFixed(2),
          quantity: parseFloat(itemQty).toFixed(2)
        }
      ];

      itemDesc = '';
      itemPrice = 1;
      itemQty = 1;

      save();
      document.getElementById('itemDesc').focus();
    } else {
      // Display gentle error message
      showError('Please enter valid item details.');
    }
  }

  function deleteItem(index) {
    appState.items = appState.items.filter((_, i) => i !== index);
    save();
  }

  function reset() {
    appState = {
      company: { name: '', logo: '' },
      invoice: {
        number: '',
        created: '',
        due: '',
        from: '',
        fromDetailsVisible: false,
        to: '',
        toDetailsVisible: false
      },
      items: [],
      taxPercent: '',
      discountPercent: '',
      currency: 'USD'
    };

    save();
  }

  onMount(() => {
    const savedData = localStorage.getItem('invoiceData');
    if (savedData) {
      appState = JSON.parse(savedData);
    } else {
      reset();
    }

    // Intelligent defaults
    if (!appState.invoice.number) {
      appState.invoice.number = '#' + Math.floor(Math.random() * 90000 + 10000);
    }
    if (!appState.invoice.created) {
      appState.invoice.created = new Date().toISOString().split('T')[0];
    }
    if (!appState.invoice.due) {
      const dueDate = new Date();
      dueDate.setDate(dueDate.getDate() + 30);
      appState.invoice.due = dueDate.toISOString().split('T')[0];
    }
  });

  let itemDesc = '';
  let itemPrice = 1;
  let itemQty = 1;

  let showForm = false;
  let showErrorMsg = false;
  let errorMsg = '';

  function showError(message) {
    errorMsg = message;
    showErrorMsg = true;
    setTimeout(() => (showErrorMsg = false), 3000);
  }

  $: subTotal = appState.items
    .reduce((total, item) => {
      return total + item.price * item.quantity;
    }, 0)
    .toFixed(2);

  $: discountAmount = ((subTotal * (+appState.discountPercent || 0)) / 100).toFixed(2);
  $: taxableAmount = (subTotal - discountAmount).toFixed(2);
  $: taxAmount = ((taxableAmount * (+appState.taxPercent || 0)) / 100).toFixed(2);
  $: totalDue = (parseFloat(taxableAmount) + parseFloat(taxAmount)).toFixed(2);

  let isDiscountEnabled = appState.discountPercent !== '';
  let isTaxEnabled = appState.taxPercent !== '';

  // For instant preview toggle
  let showPreview = false;

  // Handle company logo upload
  function handleImageChange(event) {
    const file = event.target.files[0];

    if (file && ['image/png', 'image/jpeg', 'image/webp'].includes(file.type)) {
      const reader = new FileReader();
      reader.onload = () => {
        appState.company.logo = reader.result;
        save();
      };
      reader.readAsDataURL(file);
    } else {
      showError('Please select a valid image (.png, .jpg, or .webp)');
    }
  }

  // Handle form expansion
  async function handleInitialInput() {
    if (initialInput.trim() !== '') {
      appState.items = [
        {
          desc: initialInput.trim(),
          price: '0.00',
          quantity: '1'
        }
      ];
      showForm = true;
      await tick();
      document.getElementById('companyName').focus();
    } else {
      showError('Please enter what you are invoicing for.');
    }
  }

  let initialInput = '';
</script>

<svelte:window bind:innerWidth />

<div class="app-container">
  <!-- One-Field Start -->
  {#if !showForm}
    <div class="initial-input-container">
      <h1>Create an Invoice</h1>
      <div class="initial-input">
        <input
          type="text"
          bind:value={initialInput}
          placeholder="What are you invoicing for?"
          on:keypress={(e) => e.key === 'Enter' && handleInitialInput()}
        />
        <button on:click={handleInitialInput}>Start</button>
      </div>
    </div>
  {/if}

  <!-- Main Form -->
  {#if showForm}
    <div class="main-container">
      <!-- Edit and Preview Panels -->
      <div class="panels">
        <!-- Edit Panel -->
        <div class="edit-panel">
          <!-- Smart Company Section -->
          <section class="company-section">
            <input
              id="companyName"
              type="text"
              bind:value={appState.company.name}
              placeholder="Company Name"
              class="company-name"
            />
            {#if appState.company.logo}
              <div class="logo-wrapper">
                <img src={appState.company.logo} alt="Company Logo" class="company-logo" />
                <button class="remove-logo" on:click={() => { appState.company.logo = ''; save(); }}>
                  <Trash2 />
                </button>
              </div>
            {:else}
              <label class="logo-upload">
                <ImagePlus />
                <span>Upload Logo</span>
                <input type="file" accept=".png,.jpg,.jpeg,.webp" on:change={handleImageChange} />
              </label>
            {/if}
          </section>

          <!-- Invoice Details -->
          <section class="invoice-details">
            <div class="input-group">
              <label>Invoice Number</label>
              <input type="text" bind:value={appState.invoice.number} />
            </div>
            <div class="input-group">
              <label>Invoice Date</label>
              <input type="date" bind:value={appState.invoice.created} />
            </div>
            <div class="input-group">
              <label>Due Date</label>
              <input type="date" bind:value={appState.invoice.due} />
            </div>
            <div class="input-group">
              <label>Currency</label>
              <select bind:value={appState.currency}>
                {#each currencyOptions as option}
                  <option value={option.code}>{option.flag} {option.code} - {option.name}</option>
                {/each}
              </select>
            </div>
          </section>

          <!-- Intuitive Client Input -->
          <section class="client-section">
            <div class="client-input">
              <input
                type="text"
                bind:value={appState.invoice.to}
                placeholder="Bill To"
                on:focus={() => appState.invoice.toDetailsVisible = true}
              />
              {#if appState.invoice.toDetailsVisible}
                <div class="client-details" transition:fade>
                  <!-- Additional client fields can be added here -->
                </div>
              {/if}
            </div>
          </section>

          <!-- Dynamic Line Items -->
          <section class="items-section">
            <h2>Invoice Items</h2>
            <form class="item-form" on:submit|preventDefault={addItem}>
              <input
                id="itemDesc"
                type="text"
                bind:value={itemDesc}
                placeholder="Description"
                required
              />
              <input
                type="number"
                bind:value={itemPrice}
                placeholder="Unit Price"
                min="0"
                step="0.01"
                required
              />
              <input
                type="number"
                bind:value={itemQty}
                placeholder="Quantity"
                min="1"
                step="1"
                required
              />
              <button type="submit" class="add-item">
                <Plus />
              </button>
            </form>

            <div class="items-table">
              {#each appState.items as item, index}
                <div class="table-row">
                  <div>
                    <input
                      type="text"
                      bind:value={item.desc}
                      on:input={() => save()}
                      required
                    />
                  </div>
                  <div>
                    <input
                      type="number"
                      bind:value={item.price}
                      on:input={() => save()}
                      min="0"
                      step="0.01"
                      required
                    />
                  </div>
                  <div>
                    <input
                      type="number"
                      bind:value={item.quantity}
                      on:input={() => save()}
                      min="1"
                      step="1"
                      required
                    />
                  </div>
                  <div>{(item.price * item.quantity).toFixed(2)}</div>
                  <div>
                    <button class="delete-item" on:click={() => deleteItem(index)}>
                      <Trash2 />
                    </button>
                  </div>
                </div>
              {/each}
            </div>
          </section>

          <!-- Totals -->
          <section class="totals-section">
            <div class="totals-row">
              <span>Subtotal</span>
              <span>{subTotal}</span>
            </div>
            <div class="totals-row">
              <span>
                Discount (%)
                <label class="toggle-switch">
                  <input type="checkbox" bind:checked={isDiscountEnabled} />
                  <span class="slider"></span>
                </label>
              </span>
              <span>
                <input
                  type="number"
                  bind:value={appState.discountPercent}
                  on:input={() => save()}
                  min="0"
                  max="100"
                  step="0.01"
                  disabled={!isDiscountEnabled}
                />
              </span>
            </div>
            <div class="totals-row">
              <span>Taxable Amount</span>
              <span>{taxableAmount}</span>
            </div>
            <div class="totals-row">
              <span>
                Tax (%)
                <label class="toggle-switch">
                  <input type="checkbox" bind:checked={isTaxEnabled} />
                  <span class="slider"></span>
                </label>
              </span>
              <span>
                <input
                  type="number"
                  bind:value={appState.taxPercent}
                  on:input={() => save()}
                  min="0"
                  max="100"
                  step="0.01"
                  disabled={!isTaxEnabled}
                />
              </span>
            </div>
            <div class="totals-row total-due">
              <span>Total Due</span>
              <span>{totalDue}</span>
            </div>
          </section>
        </div>

        <!-- Preview Panel -->
        {#if innerWidth > 768}
          <div class="preview-panel">
            <!-- Implement a simplified preview of the invoice -->
            <div class="invoice-preview">
              <h2>Invoice Preview</h2>
              <p><strong>{appState.company.name}</strong></p>
              <p>{appState.invoice.number}</p>
              <p>{appState.invoice.created}</p>
              <!-- Additional preview details -->
            </div>
          </div>
        {/if}
      </div>

      <!-- Quick Actions Footer -->
      <footer class="quick-actions">
        <button on:click={() => window.print()}>
          <Download />
          <span>Download PDF</span>
        </button>
        <!-- Additional buttons like "Send Invoice" and "Save as Link" can be added -->
      </footer>
    </div>
  {/if}

  <!-- Error Message -->
  {#if showErrorMsg}
    <div class="error-message" transition:fade>
      {errorMsg}
    </div>
  {/if}
</div>

<style>
  /* Global Styles */
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&display=swap');

  body {
    margin: 0;
    font-family: 'Inter', sans-serif;
    background-color: #f9fafb;
    color: #111827;
  }

  .app-container {
    max-width: 960px;
    margin: 0 auto;
    padding: 20px;
  }

  /* One-Field Start */
  .initial-input-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 100px 20px;
  }

  .initial-input-container h1 {
    font-size: 2rem;
    margin-bottom: 20px;
  }

  .initial-input {
    display: flex;
    align-items: center;
    gap: 10px;
  }

  .initial-input input {
    padding: 12px;
    font-size: 1rem;
    width: 300px;
    border: 1px solid #d1d5db;
    border-radius: 6px;
  }

  .initial-input button {
    padding: 12px 20px;
    background-color: #3b82f6;
    color: #fff;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    transition: background-color 0.2s;
  }

  .initial-input button:hover {
    background-color: #2563eb;
  }

  /* Main Container */
  .main-container {
    margin-top: 40px;
  }

  .panels {
    display: flex;
  }

  .edit-panel {
    flex: 1;
    padding-right: 20px;
  }

  .preview-panel {
    width: 40%;
    padding-left: 20px;
    border-left: 1px solid #e5e7eb;
  }

  /* Company Section */
  .company-section {
    margin-bottom: 40px;
  }

  .company-name {
    font-size: 1.5rem;
    font-weight: 700;
    border: none;
    border-bottom: 1px solid #d1d5db;
    padding: 8px 0;
    width: 100%;
    transition: border-color 0.2s;
  }

  .company-name:focus {
    outline: none;
    border-color: #3b82f6;
  }

  .logo-wrapper {
    position: relative;
    margin-top: 20px;
  }

  .company-logo {
    max-height: 80px;
  }

  .remove-logo {
    position: absolute;
    top: -10px;
    right: -10px;
    background: #fff;
    border: none;
    cursor: pointer;
    padding: 4px;
    border-radius: 50%;
    transition: background 0.2s;
  }

  .remove-logo:hover {
    background: #f3f4f6;
  }

  .logo-upload {
    display: flex;
    align-items: center;
    cursor: pointer;
    color: #6b7280;
    margin-top: 20px;
  }

  .logo-upload input {
    display: none;
  }

  .logo-upload:hover {
    color: #111827;
  }

  /* Invoice Details */
  .invoice-details {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    margin-bottom: 40px;
  }

  .invoice-details .input-group {
    flex: 1 1 200px;
  }

  .invoice-details label {
    font-size: 0.9rem;
    color: #6b7280;
    margin-bottom: 4px;
  }

  .invoice-details input,
  .invoice-details select {
    width: 100%;
    padding: 8px;
    border: 1px solid #d1d5db;
    border-radius: 6px;
    font-size: 1rem;
    transition: border-color 0.2s;
  }

  .invoice-details input:focus,
  .invoice-details select:focus {
    outline: none;
    border-color: #3b82f6;
  }

  /* Client Section */
  .client-section {
    margin-bottom: 40px;
  }

  .client-input input {
    width: 100%;
    padding: 12px;
    border: 1px solid #d1d5db;
    border-radius: 6px;
    font-size: 1rem;
    transition: border-color 0.2s;
  }

  .client-input input:focus {
    outline: none;
    border-color: #3b82f6;
  }

  /* Items Section */
  .items-section {
    margin-bottom: 40px;
  }

  .items-section h2 {
    font-size: 1.25rem;
    font-weight: 600;
    margin-bottom: 20px;
  }

  .item-form {
    display: flex;
    gap: 10px;
    margin-bottom: 20px;
  }

  .item-form input {
    flex: 1;
    padding: 8px;
    border: 1px solid #d1d5db;
    border-radius: 6px;
    font-size: 1rem;
    transition: border-color 0.2s;
  }

  .item-form input:focus {
    outline: none;
    border-color: #3b82f6;
  }

  .add-item {
    background: none;
    border: none;
    cursor: pointer;
    color: #6b7280;
    transition: color 0.2s;
    padding: 0;
  }

  .add-item:hover {
    color: #111827;
  }

  .items-table .table-row {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 10px;
  }

  .items-table .table-row div {
    flex: 1;
  }

  .items-table .table-row input {
    width: 100%;
    padding: 8px;
    border: 1px solid #d1d5db;
    border-radius: 6px;
    font-size: 1rem;
    transition: border-color 0.2s;
  }

  .items-table .table-row input:focus {
    outline: none;
    border-color: #3b82f6;
  }

  .delete-item {
    background: none;
    border: none;
    cursor: pointer;
    color: #6b7280;
    transition: color 0.2s;
    padding: 0;
  }

  .delete-item:hover {
    color: #111827;
  }

  /* Totals Section */
  .totals-section {
    max-width: 400px;
    margin-left: auto;
  }

  .totals-row {
    display: flex;
    justify-content: space-between;
    margin-bottom: 10px;
    font-size: 1rem;
  }

  .totals-row input {
    width: 80px;
    padding: 8px;
    border: 1px solid #d1d5db;
    border-radius: 6px;
    text-align: right;
    transition: border-color 0.2s;
  }

  .totals-row input:focus {
    outline: none;
    border-color: #3b82f6;
  }

  .totals-row.total-due {
    font-weight: 700;
  }

  .toggle-switch {
    position: relative;
    display: inline-block;
    width: 36px;
    height: 20px;
    margin-left: 8px;
  }

  .toggle-switch input {
    opacity: 0;
    width: 0;
    height: 0;
  }

  .toggle-switch .slider {
    position: absolute;
    cursor: pointer;
    background-color: #d1d5db;
    border-radius: 20px;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    transition: background-color 0.2s;
  }

  .toggle-switch .slider:before {
    position: absolute;
    content: '';
    height: 16px;
    width: 16px;
    left: 2px;
    bottom: 2px;
    background-color: #fff;
    border-radius: 50%;
    transition: transform 0.2s;
  }

  .toggle-switch input:checked + .slider {
    background-color: #3b82f6;
  }

  .toggle-switch input:checked + .slider:before {
    transform: translateX(16px);
  }

  /* Quick Actions Footer */
  .quick-actions {
    display: flex;
    justify-content: center;
    margin-top: 40px;
  }

  .quick-actions button {
    background-color: #3b82f6;
    color: #fff;
    border: none;
    padding: 12px 24px;
    font-size: 1rem;
    border-radius: 6px;
    cursor: pointer;
    display: inline-flex;
    align-items: center;
    gap: 8px;
    transition: background-color 0.2s;
  }

  .quick-actions button:hover {
    background-color: #2563eb;
  }

  /* Error Message */
  .error-message {
    position: fixed;
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
    background-color: #f87171;
    color: #fff;
    padding: 12px 24px;
    border-radius: 6px;
    font-size: 1rem;
  }

  /* Responsive Design */
  @media (max-width: 768px) {
    .panels {
      flex-direction: column;
    }

    .preview-panel {
      display: none;
    }
  }

  /* Print Styles */
  @media print {
    .app-container {
      padding: 0;
    }

    .initial-input-container,
    .quick-actions,
    .error-message {
      display: none;
    }

    .edit-panel {
      width: 100%;
      padding: 0;
    }

    input,
    textarea,
    select {
      border: none;
    }
  }
</style>
