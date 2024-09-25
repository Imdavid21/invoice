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
    { code: 'EUR', symbol: '€', name: 'Euro', flag: '🇪🇺' }
    // Add more currencies here if needed
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

  // Declare innerWidth for binding
  let innerWidth;

  // Svelte will automatically bind window.innerWidth
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
  /* Your CSS styles */
</style>
