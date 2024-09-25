<script>
  import { Plus, Trash, ImagePlus, RotateCw, Download } from 'lucide-svelte';
  import { onMount } from 'svelte';

  let appState = {
    company: { name: '', logo: '' },
    invoice: {
      number: '007',
      created: '',
      due: '',
      from: '',
      fromTaxIdName: '',
      fromTaxId: '',
      fromContact: { mail: '', phone: '' },
      to: '',
      toTaxIdName: '',
      toTaxId: '',
      toContact: { mail: '', phone: '' }
    },
    items: [{ desc: '', price: '', quantity: '' }],
    taxPercent: '',
    discountPercent: '',
    note: '',
    paymentFields: [
      { label: 'Account No.', value: '' },
      { label: 'Account Name', value: '' },
      { label: 'Bank Name', value: '' },
      { label: 'IFSC', value: '' },
      { label: 'SWIFT Code', value: '' }
    ],
    currency: 'USD'
  };

  let currencyOptions = [
    { code: 'USD', symbol: '$', name: 'US Dollar', flag: '🇺🇸' },
    { code: 'EUR', symbol: '€', name: 'Euro', flag: '🇪🇺' },
    { code: 'GBP', symbol: '£', name: 'British Pound', flag: '🇬🇧' },
    { code: 'JPY', symbol: '¥', name: 'Japanese Yen', flag: '🇯🇵' },
    { code: 'AUD', symbol: 'A$', name: 'Australian Dollar', flag: '🇦🇺' },
    { code: 'CAD', symbol: 'C$', name: 'Canadian Dollar', flag: '🇨🇦' },
    { code: 'CHF', symbol: 'CHF', name: 'Swiss Franc', flag: '🇨🇭' },
    { code: 'CNY', symbol: '¥', name: 'Chinese Yuan', flag: '🇨🇳' },
    { code: 'INR', symbol: '₹', name: 'Indian Rupee', flag: '🇮🇳' },
    { code: 'RUB', symbol: '₽', name: 'Russian Ruble', flag: '🇷🇺' }
  ];

  function getCurrencySymbol(code) {
    const currency = currencyOptions.find((c) => c.code === code);
    return currency ? currency.symbol : '';
  }

  function handleImageChange(event) {
    const file = event.target.files[0];

    if (file && (file.type === 'image/png' || file.type === 'image/jpeg' || file.type === 'image/webp')) {
      const reader = new FileReader();
      reader.onload = () => {
        appState.company.logo = reader.result;
        save();
      };
      reader.readAsDataURL(file);
    } else {
      alert('Please select a valid image (.png, .jpg, or .webp)');
    }
  }

  function save() {
    // No localStorage interaction as per current needs
  }

  function addItem() {
    if (itemDesc != '' && itemPrice > 0 && itemQty > 0) {
      appState.items = [
        ...appState.items,
        {
          desc: itemDesc,
          price: parseFloat(itemPrice).toFixed(2),
          quantity: parseFloat(itemQty).toFixed(2)
        }
      ];

      itemDesc = '';
      itemPrice = 1;
      itemQty = 1;

      save();
      document.getElementById('descBox').focus();
    }
  }

  function deleteItem(index) {
    appState.items = appState.items.filter((_, i) => i !== index);
    save();
  }

  function reset() {
    appState = {
      company: { name: 'Wayne Enterprises', logo: '' },
      invoice: {
        number: '007',
        created: '',
        due: '',
        from: '',
        fromTaxIdName: 'Enter Tax Name',
        fromTaxId: '',
        fromContact: { mail: '', phone: '' },
        to: '',
        toTaxIdName: 'Enter Tax Name',
        toTaxId: '',
        toContact: { mail: '', phone: '' }
      },
      items: [
        { desc: 'Batmobile Repair', price: '15000', quantity: '1' },
        { desc: 'Grappling Hooks', price: '500', quantity: '20' },
        { desc: 'Interdimensional Portal Gun', price: '200000', quantity: '1' }
      ],
      taxPercent: '',
      discountPercent: '',
      note: '',
      paymentFields: [
        { label: 'Account No.', value: '' },
        { label: 'Account Name', value: '' },
        { label: 'Bank Name', value: '' },
        { label: 'IFSC', value: '' },
        { label: 'SWIFT Code', value: '' }
      ],
      currency: 'USD'
    };

    save();
  }

  onMount(() => {
    reset();
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
  let isDiscountEnabled = true;
  let isTaxEnabled = true;
</script>

<div
  class="max-w-screen-md mx-auto px-6 py-8 flex flex-col space-y-6 font-montserrat bg-[#FAFAFA] border border-[#E2E2E2] rounded-xl shadow-sm print:shadow-none print:bg-white print:border-none"
>
  <!-- Currency, Created, and Due Dates Reverted to Original Text Inputs -->
  <div class="flex flex-row justify-between items-center gap-4">
    <select
      bind:value={appState.currency}
      class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] flex-1"
    >
      {#each currencyOptions as option}
        <option value={option.code}>
          {option.flag} {option.name}
        </option>
      {/each}
    </select>

    <input
      type="text"
      bind:value={appState.invoice.created}
      placeholder="Date Created"
      class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] flex-1"
    />

    <input
      type="text"
      bind:value={appState.invoice.due}
      placeholder="Due Date"
      class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] flex-1"
    />
  </div>

  <!-- From & To Section with Vertical Alignment of Contact Details -->
  <div class="flex flex-row gap-6">
    <div class="flex-1">
      <h4 class="mb-2 font-bold">From</h4>
      <textarea
        style="resize: none; padding: 8px 12px;"
        placeholder="Enter sender's address"
        cols="30"
        rows="3"
        maxlength="150"
        bind:value={appState.invoice.from}
        class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
      ></textarea>
      <div class="flex flex-col gap-1 mt-2">
        <label class="text-sm">Tax Name:</label>
        <input
          type="text"
          bind:value={appState.invoice.fromTaxIdName}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
        />
        <label class="text-sm">Tax ID:</label>
        <input
          type="text"
          bind:value={appState.invoice.fromTaxId}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
        />
        <label class="text-sm">Email:</label>
        <input
          type="email"
          bind:value={appState.invoice.fromContact.mail}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
        />
        <label class="text-sm">Phone:</label>
        <input
          type="text"
          bind:value={appState.invoice.fromContact.phone}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
        />
      </div>
    </div>

    <div class="flex-1">
      <h4 class="mb-2 font-bold">To</h4>
      <textarea
        style="resize: none; padding: 8px 12px;"
        placeholder="Enter recipient's address"
        cols="30"
        rows="3"
        maxlength="150"
        bind:value={appState.invoice.to}
        class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
      ></textarea>
      <div class="flex flex-col gap-1 mt-2">
        <label class="text-sm">Tax Name:</label>
        <input
          type="text"
          bind:value={appState.invoice.toTaxIdName}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
        />
        <label class="text-sm">Tax ID:</label>
        <input
          type="text"
          bind:value={appState.invoice.toTaxId}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
        />
        <label class="text-sm">Email:</label>
        <input
          type="email"
          bind:value={appState.invoice.toContact.mail}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
        />
        <label class="text-sm">Phone:</label>
        <input
          type="text"
          bind:value={appState.invoice.toContact.phone}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
        />
      </div>
    </div>
  </div>

  <!-- Original Item Entry Section -->
  <form class="flex flex-row justify-between gap-2">
    <button
      disabled={itemDesc == ''}
      on:click={() => addItem()}
      class="p-2 rounded-lg bg-[#E2E2E2] hover:bg-[#CFCFCF] transition-all duration-100 ease-in-out print:hidden"
    >
      <Plus stroke="#557571" />
    </button>

    <input
      id="descBox"
      type="text"
      bind:value={itemDesc}
      placeholder="Item description"
      class="border border-[#E2E2E2] rounded-lg p-3 focus:outline-none focus:border-[#5A5A5A] grow"
      on:keypress={(e) => e.key == 'Enter' && addItem()}
    />
    <input
      type="number"
      bind:value={itemPrice}
      placeholder="Unit Price"
      min="0"
      step="0.01"
      class="border border-[#E2E2E2] rounded-lg p-3 focus:outline-none focus:border-[#5A5A5A] w-32"
      on:keypress={(e) => e.key == 'Enter' && addItem()}
    />
    <input
      type="number"
      bind:value={itemQty}
      placeholder="Quantity"
      min="1"
      step="1"
      class="border border-[#E2E2E2] rounded-lg p-3 focus:outline-none focus:border-[#5A5A5A] w-32"
      on:keypress={(e) => e.key == 'Enter' && addItem()}
    />
    <p class="border border-[#E2E2E2] p-3 w-32 text-right">
      {getCurrencySymbol(appState.currency)}{(itemPrice * itemQty).toFixed(2)}
    </p>
  </form>

  <!-- Download Button -------------------------------------------------->
  <div class="mt-6 flex justify-center print:hidden">
    <button
      on:click={() => window.print()}
      class="px-4 py-3 rounded-lg bg-[#E2E2E2] text-[#333] font-semibold flex items-center gap-2 hover:bg-[#CFCFCF] transition-transform duration-150"
    >
      <Download class="w-5 h-5" />
      <span>Download Invoice</span>
    </button>
  </div>
</div>

<style>
  .toggle-switch {
    width: 36px;
    height: 18px;
    background-color: #e2e2e2;
    border-radius: 20px;
    position: relative;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  .toggle-switch:before {
    content: '';
    position: absolute;
    top: 1px;
    left: 1px;
    width: 16px;
    height: 16px;
    background-color: #8eaccd;
    border-radius: 50%;
    transition: all 0.3s;
  }

  .toggle-switch.active {
    background-color: #8eaccd;
  }

  .toggle-switch.active:before {
    transform: translateX(18px);
    background-color: #feffd9;
  }
</style>
