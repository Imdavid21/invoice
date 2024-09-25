<script>
  import { Plus, Trash2, ImagePlus, Download } from 'lucide-svelte';
  import { onMount } from 'svelte';

  let appState = {
    company: { name: '', logo: '' },
    invoice: {
      number: '#0001',
      created: '',
      due: '',
      from: '',
      fromTaxIdName: 'Tax Name',
      fromTaxId: '',
      fromContact: { mail: '', phone: '' },
      to: '',
      toTaxIdName: 'Tax Name',
      toTaxId: '',
      toContact: { mail: '', phone: '' }
    },
    items: [],
    taxPercent: '',
    discountPercent: '',
    note: '',
    payment: {
      accountNumber: '',
      accountName: '',
      bank: '',
      ifsc: '',
      swiftCode: ''
    },
    currency: 'USD'
  };

  let currencyOptions = [
    { code: 'USD', symbol: '$', name: 'US Dollar', flag: '🇺🇸' },
    { code: 'EUR', symbol: '€', name: 'Euro', flag: '🇪🇺' },
    { code: 'GBP', symbol: '£', name: 'British Pound', flag: '🇬🇧' },
    { code: 'JPY', symbol: '¥', name: 'Japanese Yen', flag: '🇯🇵' },
    { code: 'AUD', symbol: 'A$', name: 'Australian Dollar', flag: '🇦🇺' },
    // ... add more currencies as needed
  ];

  function getCurrencySymbol(code) {
    const currency = currencyOptions.find((c) => c.code === code);
    return currency ? currency.symbol : '';
  }

  let loadingLogo = false;

  function handleImageChange(event) {
    const file = event.target.files[0];

    if (file && ['image/png', 'image/jpeg', 'image/webp'].includes(file.type)) {
      loadingLogo = true;
      const reader = new FileReader();
      reader.onload = () => {
        appState.company.logo = reader.result;
        loadingLogo = false;
        save();
      };
      reader.readAsDataURL(file);
    } else {
      alert('Please select a valid image (.png, .jpg, or .webp)');
    }
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
      document.getElementById('descBox').focus();
    } else {
      alert('Please enter valid item details.');
    }
  }

  function deleteItem(index) {
    appState.items = appState.items.filter((_, i) => i !== index);
    save();
  }

  function reset() {
    appState = {
      company: { name: 'Your Company Name', logo: '' },
      invoice: {
        number: '#0001',
        created: '',
        due: '',
        from: '',
        fromTaxIdName: 'Tax Name',
        fromTaxId: '',
        fromContact: { mail: '', phone: '' },
        to: '',
        toTaxIdName: 'Tax Name',
        toTaxId: '',
        toContact: { mail: '', phone: '' }
      },
      items: [],
      taxPercent: '',
      discountPercent: '',
      note: '',
      payment: {
        accountNumber: '',
        accountName: '',
        bank: '',
        ifsc: '',
        swiftCode: ''
      },
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
  });

  let itemDesc = '';
  let itemPrice = 1;
  let itemQty = 1;

  $: subTotal = appState.items
    .reduce((total, item) => {
      return total + item.price * item.quantity;
    }, 0)
    .toFixed(2);

  $: discountAmount = ((subTotal * (+appState.discountPercent || 0)) / 100).toFixed(2);
  $: taxableAmount = (subTotal - discountAmount).toFixed(2);
  $: taxAmount = ((taxableAmount * (+appState.taxPercent || 0)) / 100).toFixed(2);
  $: totalDue = (parseFloat(taxableAmount) + parseFloat(taxAmount)).toFixed(2);

  // Toggles
  let isDiscountEnabled = appState.discountPercent !== '';
  let isTaxEnabled = appState.taxPercent !== '';
</script>

<svelte:head>
  <link rel="stylesheet" href="https://rsms.me/inter/inter.css" />
</svelte:head>

<div class="max-w-4xl mx-auto p-6 bg-white dark:bg-gray-900 rounded-lg shadow-lg">
  <!-- Header -->
  <header class="flex flex-wrap justify-between items-center mb-8">
    <div class="flex items-center space-x-4">
      {#if appState.company.logo}
        <div class="relative">
          <img
            src={appState.company.logo}
            alt="Company Logo"
            class="h-16 w-auto object-contain"
          />
          <button
            class="absolute -top-2 -right-2 bg-red-500 text-white rounded-full p-1 hover:bg-red-600"
            on:click={() => {
              appState.company.logo = '';
              save();
            }}
            aria-label="Remove Logo"
          >
            <Trash2 class="h-4 w-4" />
          </button>
        </div>
      {:else}
        <label class="flex items-center space-x-2 cursor-pointer text-blue-600 hover:text-blue-700">
          <ImagePlus class="h-6 w-6" />
          <span class="text-sm font-medium">Upload Logo</span>
          <input
            type="file"
            accept=".png,.jpg,.jpeg,.webp"
            on:change={handleImageChange}
            class="hidden"
          />
        </label>
      {/if}
      <input
        type="text"
        bind:value={appState.company.name}
        placeholder="Your Company Name"
        class="text-2xl font-bold border-none focus:ring-0 dark:bg-gray-900 dark:text-white"
      />
    </div>
    <div class="mt-4 md:mt-0">
      <h1 class="text-3xl font-extrabold text-gray-900 dark:text-white">Invoice</h1>
      <div class="mt-2 grid grid-cols-1 gap-2 sm:grid-cols-2">
        <div>
          <label class="block text-sm text-gray-700 dark:text-gray-300">Invoice Number</label>
          <input
            type="text"
            bind:value={appState.invoice.number}
            class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
          />
        </div>
        <div>
          <label class="block text-sm text-gray-700 dark:text-gray-300">Date</label>
          <input
            type="date"
            bind:value={appState.invoice.created}
            class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
          />
        </div>
        <div>
          <label class="block text-sm text-gray-700 dark:text-gray-300">Due Date</label>
          <input
            type="date"
            bind:value={appState.invoice.due}
            class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
          />
        </div>
        <div>
          <label class="block text-sm text-gray-700 dark:text-gray-300">Currency</label>
          <select
            bind:value={appState.currency}
            class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
          >
            {#each currencyOptions as option}
              <option value={option.code}>
                {option.flag} {option.code} - {option.name}
              </option>
            {/each}
          </select>
        </div>
      </div>
    </div>
  </header>

  <hr class="mb-8 border-gray-200 dark:border-gray-700" />

  <!-- Addresses -->
  <section class="grid grid-cols-1 gap-8 md:grid-cols-2 mb-8">
    <div>
      <h2 class="text-xl font-semibold text-gray-900 dark:text-white mb-4">Bill From</h2>
      <textarea
        bind:value={appState.invoice.from}
        rows="4"
        placeholder="Your address"
        class="block w-full border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
      ></textarea>
      <div class="mt-4 grid grid-cols-1 gap-4 sm:grid-cols-2">
        <div>
          <label class="block text-sm text-gray-700 dark:text-gray-300">
            {appState.invoice.fromTaxIdName}
          </label>
          <input
            type="text"
            bind:value={appState.invoice.fromTaxId}
            class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
          />
        </div>
        <div>
          <label class="block text-sm text-gray-700 dark:text-gray-300">Email</label>
          <input
            type="email"
            bind:value={appState.invoice.fromContact.mail}
            class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
          />
        </div>
        <div>
          <label class="block text-sm text-gray-700 dark:text-gray-300">Phone</label>
          <input
            type="tel"
            bind:value={appState.invoice.fromContact.phone}
            class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
          />
        </div>
      </div>
    </div>
    <div>
      <h2 class="text-xl font-semibold text-gray-900 dark:text-white mb-4">Bill To</h2>
      <textarea
        bind:value={appState.invoice.to}
        rows="4"
        placeholder="Client's address"
        class="block w-full border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
      ></textarea>
      <div class="mt-4 grid grid-cols-1 gap-4 sm:grid-cols-2">
        <div>
          <label class="block text-sm text-gray-700 dark:text-gray-300">
            {appState.invoice.toTaxIdName}
          </label>
          <input
            type="text"
            bind:value={appState.invoice.toTaxId}
            class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
          />
        </div>
        <div>
          <label class="block text-sm text-gray-700 dark:text-gray-300">Email</label>
          <input
            type="email"
            bind:value={appState.invoice.toContact.mail}
            class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
          />
        </div>
        <div>
          <label class="block text-sm text-gray-700 dark:text-gray-300">Phone</label>
          <input
            type="tel"
            bind:value={appState.invoice.toContact.phone}
            class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
          />
        </div>
      </div>
    </div>
  </section>

  <!-- Items -->
  <section class="mb-8">
    <h2 class="text-xl font-semibold text-gray-900 dark:text-white mb-4">Invoice Items</h2>
    <form
      class="grid grid-cols-1 gap-4 sm:grid-cols-12 mb-6"
      on:submit|preventDefault={addItem}
    >
      <input
        id="descBox"
        type="text"
        bind:value={itemDesc}
        placeholder="Item Description"
        class="col-span-5 sm:col-span-5 border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
        required
      />
      <input
        type="number"
        bind:value={itemPrice}
        placeholder="Unit Price"
        min="0"
        step="0.01"
        class="col-span-3 sm:col-span-2 border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
        required
      />
      <input
        type="number"
        bind:value={itemQty}
        placeholder="Quantity"
        min="1"
        step="1"
        class="col-span-3 sm:col-span-2 border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
        required
      />
      <button
        type="submit"
        class="col-span-1 sm:col-span-1 bg-blue-600 text-white rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 flex items-center justify-center"
        aria-label="Add Item"
      >
        <Plus class="h-5 w-5" />
      </button>
    </form>

    <table class="w-full text-left table-auto mb-4">
      <thead>
        <tr class="bg-gray-100 dark:bg-gray-800">
          <th class="px-4 py-2">Description</th>
          <th class="px-4 py-2">Unit Price ({getCurrencySymbol(appState.currency)})</th>
          <th class="px-4 py-2">Quantity</th>
          <th class="px-4 py-2">Total ({getCurrencySymbol(appState.currency)})</th>
          <th class="px-4 py-2">Actions</th>
        </tr>
      </thead>
      <tbody>
        {#each appState.items as item, index}
          <tr class="border-b dark:border-gray-700">
            <td class="px-4 py-2">
              <input
                type="text"
                bind:value={item.desc}
                on:input={() => save()}
                class="w-full border-none focus:ring-0 dark:bg-gray-900 dark:text-white"
                required
              />
            </td>
            <td class="px-4 py-2">
              <input
                type="number"
                bind:value={item.price}
                on:input={() => save()}
                min="0"
                step="0.01"
                class="w-full border-none focus:ring-0 dark:bg-gray-900 dark:text-white"
                required
              />
            </td>
            <td class="px-4 py-2">
              <input
                type="number"
                bind:value={item.quantity}
                on:input={() => save()}
                min="1"
                step="1"
                class="w-full border-none focus:ring-0 dark:bg-gray-900 dark:text-white"
                required
              />
            </td>
            <td class="px-4 py-2">
              {(item.price * item.quantity).toFixed(2)}
            </td>
            <td class="px-4 py-2">
              <button
                class="text-red-500 hover:text-red-600"
                on:click={() => deleteItem(index)}
                aria-label="Delete Item"
              >
                <Trash2 class="h-5 w-5" />
              </button>
            </td>
          </tr>
        {/each}
      </tbody>
    </table>
  </section>

  <!-- Totals -->
  <section class="mb-8">
    <div class="max-w-md ml-auto">
      <div class="flex justify-between py-2">
        <span class="font-medium text-gray-700 dark:text-gray-300">Subtotal</span>
        <span class="text-gray-900 dark:text-white">{subTotal}</span>
      </div>
      <div class="flex justify-between py-2 items-center">
        <div class="flex items-center">
          <span class="font-medium text-gray-700 dark:text-gray-300">Discount (%)</span>
          <label class="ml-2 relative inline-flex items-center cursor-pointer">
            <input
              type="checkbox"
              bind:checked={isDiscountEnabled}
              class="sr-only peer"
            />
            <div
              class="w-9 h-5 bg-gray-200 peer-focus:outline-none peer-focus:ring-2 peer-focus:ring-blue-500 rounded-full peer dark:bg-gray-700 peer-checked:bg-blue-600"
            ></div>
          </label>
        </div>
        <input
          type="number"
          bind:value={appState.discountPercent}
          on:input={() => save()}
          min="0"
          max="100"
          step="0.01"
          disabled={!isDiscountEnabled}
          class="w-24 text-right border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
        />
      </div>
      <div class="flex justify-between py-2">
        <span class="font-medium text-gray-700 dark:text-gray-300">Taxable Amount</span>
        <span class="text-gray-900 dark:text-white">{taxableAmount}</span>
      </div>
      <div class="flex justify-between py-2 items-center">
        <div class="flex items-center">
          <span class="font-medium text-gray-700 dark:text-gray-300">Tax (%)</span>
          <label class="ml-2 relative inline-flex items-center cursor-pointer">
            <input
              type="checkbox"
              bind:checked={isTaxEnabled}
              class="sr-only peer"
            />
            <div
              class="w-9 h-5 bg-gray-200 peer-focus:outline-none peer-focus:ring-2 peer-focus:ring-blue-500 rounded-full peer dark:bg-gray-700 peer-checked:bg-blue-600"
            ></div>
          </label>
        </div>
        <input
          type="number"
          bind:value={appState.taxPercent}
          on:input={() => save()}
          min="0"
          max="100"
          step="0.01"
          disabled={!isTaxEnabled}
          class="w-24 text-right border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
        />
      </div>
      <div class="flex justify-between py-2 border-t mt-4 pt-4 border-gray-200 dark:border-gray-700">
        <span class="font-semibold text-lg text-gray-900 dark:text-white">Total Due</span>
        <span class="font-semibold text-lg text-gray-900 dark:text-white">{totalDue}</span>
      </div>
    </div>
  </section>

  <!-- Payment Info -->
  <section class="mb-8">
    <h2 class="text-xl font-semibold text-gray-900 dark:text-white mb-4">Payment Information</h2>
    <div class="grid grid-cols-1 gap-4 md:grid-cols-2">
      <div>
        <label class="block text-sm text-gray-700 dark:text-gray-300">Account Number</label>
        <input
          type="text"
          bind:value={appState.payment.accountNumber}
          maxlength="20"
          class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
        />
      </div>
      <div>
        <label class="block text-sm text-gray-700 dark:text-gray-300">Account Name</label>
        <input
          type="text"
          bind:value={appState.payment.accountName}
          maxlength="28"
          class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
        />
      </div>
      <div>
        <label class="block text-sm text-gray-700 dark:text-gray-300">Bank Name</label>
        <input
          type="text"
          bind:value={appState.payment.bank}
          maxlength="28"
          class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
        />
      </div>
      <div>
        <label class="block text-sm text-gray-700 dark:text-gray-300">IFSC</label>
        <input
          type="text"
          bind:value={appState.payment.ifsc}
          maxlength="11"
          class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
        />
      </div>
      <div>
        <label class="block text-sm text-gray-700 dark:text-gray-300">SWIFT Code</label>
        <input
          type="text"
          bind:value={appState.payment.swiftCode}
          maxlength="11"
          class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-white"
        />
      </div>
    </div>
  </section>

  <!-- Download Button -->
  <div class="text-center">
    <button
      on:click={() => window.print()}
      class="inline-flex items-center px-6 py-3 border border-transparent text-base font-medium rounded-md shadow-sm text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500"
    >
      <Download class="h-5 w-5 mr-2" />
      Download Invoice
    </button>
  </div>
</div>

<style>
  /* Ensure print styles are clean */
  @media print {
    .hidden-print {
      display: none !important;
    }
    body {
      background-color: #fff;
    }
    .dark {
      background-color: #fff;
      color: #000;
    }
    .dark * {
      background-color: #fff;
      color: #000;
    }
    input,
    textarea {
      border: none;
    }
    button {
      display: none;
    }
  }
</style>
