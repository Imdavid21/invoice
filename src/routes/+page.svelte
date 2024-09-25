<script>
  import { Plus, Trash, ImagePlus, Download } from 'lucide-svelte';
  import { onMount } from 'svelte';

  let appState = {
    company: { name: '', logo: '' },
    invoice: {
      number: '#0001',
      created: '',
      due: '',
      from: '',
      fromTaxIdName: '',
      fromTaxId: '',
      fromContact: { mail: '', phone: '' },
      to: '',
      toTaxIdName: '',
      toTaxId: '',
      toContact: { mail: '', phone: '' },
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
      swiftCode: '',
    },
    currency: 'USD',
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
    { code: 'RUB', symbol: '₽', name: 'Russian Ruble', flag: '🇷🇺' },
    { code: 'BRL', symbol: 'R$', name: 'Brazilian Real', flag: '🇧🇷' },
    { code: 'ZAR', symbol: 'R', name: 'South African Rand', flag: '🇿🇦' },
    { code: 'MXN', symbol: '$', name: 'Mexican Peso', flag: '🇲🇽' },
    { code: 'NZD', symbol: 'NZ$', name: 'New Zealand Dollar', flag: '🇳🇿' },
    { code: 'SGD', symbol: 'S$', name: 'Singapore Dollar', flag: '🇸🇬' },
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
    localStorage.setItem('invoiceData', JSON.stringify(appState));
  }

  function addItem() {
    if (itemDesc != '' && itemPrice > 0 && itemQty > 0) {
      appState.items = [
        ...appState.items,
        {
          desc: itemDesc,
          price: parseFloat(itemPrice).toFixed(2),
          quantity: parseFloat(itemQty).toFixed(2),
        },
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
        number: '#0001',
        created: '',
        due: '',
        from: '',
        fromTaxIdName: 'Enter Tax Name',
        fromTaxId: '',
        fromContact: { mail: '', phone: '' },
        to: '',
        toTaxIdName: 'Enter Tax Name',
        toTaxId: '',
        toContact: { mail: '', phone: '' },
      },
      items: [
        { desc: 'SEO Consultation', price: '5000', quantity: '1' },
        { desc: 'Social Media Strategy', price: '2000', quantity: '3' },
        { desc: 'Content Creation', price: '10000', quantity: '1' },
      ],
      taxPercent: '',
      discountPercent: '',
      note: '',
      payment: {
        accountNumber: '',
        accountName: '',
        bank: '',
        ifsc: '',
        swiftCode: '',
      },
      currency: 'USD',
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
  let isDiscountEnabled = true;
  let isTaxEnabled = true;
</script>

<svelte:body on:click={() => save()} />

<div class="invoice-container bg-white shadow-md rounded-lg p-6">
  <table class="w-full border-collapse">
    <thead>
      <tr class="bg-[#1F2A44] text-white">
        <th class="py-2 px-4 text-left">Item</th>
        <th class="py-2 px-4">Quantity</th>
        <th class="py-2 px-4">Rate</th>
        <th class="py-2 px-4 text-right">Amount</th>
      </tr>
    </thead>
    <tbody>
      {#each appState.items as item, index}
        <tr class="border-b">
          <td class="py-2 px-4">
            <input
              type="text"
              class="w-full border-none bg-transparent focus:outline-none"
              bind:value={item.desc}
            />
          </td>
          <td class="py-2 px-4 text-center">
            <input
              type="number"
              class="w-full border-none bg-transparent text-center focus:outline-none"
              bind:value={item.quantity}
              min="1"
            />
          </td>
          <td class="py-2 px-4 text-center">
            <input
              type="number"
              class="w-full border-none bg-transparent text-center focus:outline-none"
              bind:value={item.price}
              min="0"
              step="0.01"
            />
          </td>
          <td class="py-2 px-4 text-right">
            {getCurrencySymbol(appState.currency)}{(item.price * item.quantity).toFixed(2)}
          </td>
        </tr>
      {/each}
    </tbody>
  </table>

  <button
    on:click={() => addItem()}
    class="mt-4 text-green-500 flex items-center"
  >
    <Plus class="mr-2" /> + Line Item
  </button>
</div>

<style>
  .invoice-container {
    border: 1px solid #d1d5db;
  }

  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 1rem;
  }

  th, td {
    padding: 0.75rem;
    border-bottom: 1px solid #e5e7eb;
  }

  th {
    font-weight: bold;
    text-align: left;
  }

  tr:last-child td {
    border-bottom: none;
  }

  input {
    padding: 0.5rem;
    border: 1px solid transparent;
    border-radius: 4px;
    transition: border-color 0.3s;
  }

  input:focus {
    border-color: #6366f1;
    outline: none;
  }

  button {
    display: flex;
    align-items: center;
    font-size: 1rem;
    color: #059669;
    background: none;
    border: none;
    cursor: pointer;
    transition: color 0.3s;
  }

  button:hover {
    color: #047857;
  }
</style>
